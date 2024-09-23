---
title:  "B2C Auth in Copilot Studio, revisiting refresh tokens"
categories:
  - Copilot Studio
tags:
  - Authentication
  - B2C
---

I saw a post in teams the day I got back from a weeks leave. "Does anyone know about refresh tokens?" Of course, this was the perfect opportunity to reuse all the token knowledge from the Project Online work.

This is one of those interesting ones where there seemingly was no answer. No docs, no tutorial, not even any community questions. I guess we've got to figure it out ourselves.

This required quite a bit of lab prep. Firstly I needed a B2C tenant, thankfully there are docs for that.

I also needed to set up a protected API that I could call from my Copilot once I'd logged in. A nice and easy Azure function to tell me the time.

```c#
[Function("GetTime")]
public IActionResult Run([HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequest req)
{
    _logger.LogInformation("C# HTTP trigger function processed a request.");
    DateTime currentTime = DateTime.Now;
    return new OkObjectResult(currentTime.ToUniversalTime().ToShortTimeString());
}
```

Now came the guess work

I had a stab but thought I should just test this out with my good old friend Postman.

## Step1. Login into B2C and get an auth code


Base Url:

`https://<b2ctenantname>.b2clogin.com/<b2ctenantname>.onmicrosoft.com/<singupsignonuserflow>/oauth2/v2.0/authorize`

Params

| Key           | Value                                                                             |
| ------------- | --------------------------------------------------------------------------------- |
| client_id     | bd0f49e5-829c-4468-9f32-1043a83e5a17                                              |
| nonce         | anyRandomValue                                                                    |
| redirect_uri  | https://token.botframework.com/.auth/web/redirect                                 |
| scope         | offline_access openid https://grovealeb2cdir.onmicrosoft.com/tasks-api/tasks.read |
| response_type | code                                                                              |


Easier to construct the url in postman. Add the base URL in the URL bar and add each parameter. Ensure this is a GET request

![Code Request URL](/assets/b2c-copilot-studio/Pasted%20image%2020240828113132.png)

Copy the full URL from the bar (this will have all the params appended)

![Copy URL](/assets/b2c-copilot-studio/Pasted%20image%2020240828113431.png)

example URL

```
https://grovealeb2cdir.b2clogin.com/grovealeb2cdir.onmicrosoft.com/B2C_1_susi/oauth2/v2.0/authorize?client_id=bd0f49e5-829c-4468-9f32-1043a83e5a17&nonce=anyRandomValue&redirect_uri=https://token.botframework.com/.auth/web/redirect&scope=offline_access openid https://grovealeb2cdir.onmicrosoft.com/tasks-api/tasks.read&response_type=code
```

Paste the URL into the browser - You should be prompted to login to the B2C tenant

![Execute code request](/assets/b2c-copilot-studio/Pasted%20image%2020240828113603.png)


This should return the code to the bot framework URI - The browser doesn't know what to do with it but we just need the code

![Successful auth](/assets/b2c-copilot-studio/Pasted%20image%2020240828113847.png)

## 2. Authenticate using the auth code


Back to postman, a new POST request

Base URL

`https://<b2ctenantname>.b2clogin.com/<b2ctenantname>.onmicrosoft.com/<singupsignonuserflow>/oauth2/v2.0/token`


Parmas

| Key           | Value                                                                             |
| ------------- | --------------------------------------------------------------------------------- |
| client_id     | bd0f49e5-829c-4468-9f32-1043a83e5a17                                              |
| grant_type    | authorization_code                                                                |
| redirect_uri  | https://token.botframework.com/.auth/web/redirect                                 |
| scope         | offline_access openid https://grovealeb2cdir.onmicrosoft.com/tasks-api/tasks.read |
| code          | eyJraWQi ... esvg                                                                 |
| client_secret | SgF8Q~08jkmDl8...                                                                 |

Hit Send

![auth code request](/assets/b2c-copilot-studio/Pasted%20image%2020240828114540.png)

You should see a response like below. Crucially confirm the refresh token is returned as this is what Copilot studio will use the reauthenticate in the background

![Successful token request](/assets/b2c-copilot-studio/Pasted%20image%2020240828114610.png)

# Copilot Studio Config

Now that we know the auth works and we can get a refresh token from our B2C tenant, lets get that Copilot Studio config together.

TODO Add auth config details

TODO Add flow call details - inlcude commentry about use access token varaibles


## Testing

Assuming config is correct, we now need to login in - call the API, see it working and wait for the refresh token to expire. Then try again

Interestingly this all worked in my environment. However long I left it, I could always get the time from the protected API. Good news, but why was this not working for the customer?

Comparing configs with the customer, the only difference is they were not forcing login on start up. Turning off auto login did highlight that when you use the `User.AccessToken` variable the login flow is called automatically. And this is the parameter that I was sending to my flow - `User.AccessToken`. This is why mine was working, I was using this variable as an input. So each time the flow was executed the login flow would happen int he background (if logged in) or simply prompt if there was no refresh token

This still doesn't explain why it wasn't working for the customer. Rather than just comparing auth config we compared Copilots. In mine I was calling the flow from a topic whereas the customer was calling it from an action. Using the variable `User.AccessToken` as an input for an action it does not cause the login flow to be called - so if the token had expired we get an issue! 

Finally we've figured it out - But is this actually a bug?