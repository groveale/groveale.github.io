---
title:  "Connect to Project Online Site Collection using an app registration - The research"
categories:
  - SharePoint
  - Project Online
tags:
  - Authentication
---

I've been working with a customer who uses basic auth to connect to the Project Online APIs and extract project data and push it to SQL for reporting. Whilst this solution works well they are currently looking to diabled basic on the tenant, which I am pleased about. However this will now break their current data pipeline and they need a new solution to extract the data from project online that does not rely on basic auth.

Ufortunaley the Project Online APIs to not support application permission therefore out approach is to use delegated access. This is application access on behalf of a user using the applications identity.

Further details of this approach can be found here - [Get access on behalf of a user - Microsoft Graph | Microsoft Learn](https://learn.microsoft.com/en-us/graph/auth-v2-user?tabs=http)

The rest of this post is more a write up of my research on the topic. Hopefully you will find it interesting!

## Auth Method

The authentication method is OAuth 2.0 authorization code flow grant - This is not basic auth so will still function once basic auth is disabled. Further detail of the authorization method can be found here - [Microsoft identity platform and OAuth 2.0 authorization code flow | Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity-platform/v2-oauth2-auth-code-flow)

> [!NOTE]  
> This method will still require a user account that has access to the Project Online Site Collections. 

## Steps to test

1. We require an app registration in Azure AD with the following delegated SPO permission

![app permissions](/assets/project-online/Pasted%20image%2020231102102235.png)

The app requires a redirect URI - This URI will be where the code is returned that is required to authorize the on behalf of (delegated) application connection

![redirect URI](/assets/project-online/Pasted%20image%2020231102102303.png)

For testing purposes the above can be used - This will enable the auth code to be returned to a browser

`https://oauth.pstmn.io/v1/browser-callback`


2. Execute the following HTTP request in a browser (if authenticated you see no prompt). This browser session needs to be authenticated by the user that has access to the Project Online Site Collection.

```
https://login.microsoftonline.com/936f0468-e5bb-4846-b365-5ae3790deadf/oauth2/v2.0/authorize
  ?client_id=afdbe70d-fbb8-47fb-9348-ad43fed0cbda
  &response_type=code
  &redirect_uri=https://oauth.pstmn.io/v1/browser-callback 
  &response_mode=query&scope=profile openid email https://graph.microsoft.com/EnterpriseResource.Read https://graph.microsoft.com/Project.Read https://graph.microsoft.com/ProjectWebApp.FullControl https://graph.microsoft.com/ProjectWebAppReporting.Read https://graph.microsoft.com/User.Read offline_access&state=12345
```

> [!NOTE]  
> client_id should be updated to reflect the app registration that was created in the previous step

Once authenticated, you will be redirected to the redirect URI

![redirected](/assets/project-online/Pasted%20image%2020231102103015.png)

In the URL you will have something similar to the below:

```
https://oauth.pstmn.io/v1/browser-callback?code=0.AXwAaARvk7vlRkizZVrjeQ3q3w3n26-4-_tHk0itQ_7Qy9q7AKM.AgABAAIAAAAtyolDObpQQ5VtlI4uGjEPAgDs_wUA9P8lE2r45aczwmR0G3_abfM75HIR5yK_cPHIQz1NnUxaawKCcu8mw4jrFmNllDCyzxx5CKLhdDh-vnAoDNR0dq34_tLY96jtPevLvSH1dOorDLNqOjaZi7k2_mrnsFxADsd2ExkSQrJ3PxUjUnPCChn52r10YXJ9P_GP6PmlI_fkQfNNovC2yQMw5OO9bkYVmnUfiRU0Hhq4LmPSVcH9oTSrWaEsSC9js4ZLpMUIbolo_EaXIKfxEprpeJZ0tXKbqJizQqqRjnqOcDBRWMpBS-xBHPgSovV5bchlultczfu5A107-d0sfLkUyOe7tqahXJFOKrTFKG2IIzCFB2OfPNp0qbc42aEq2PUw6wz7kSVgXWYAg3hX3Jo3HQi7_3bsK7aU_q1SWhB-59Sevh3dfcFB4rvLKAEP13fi5H71G-5eC7X7jlat9ix6fQ2qzXc0aA3NopqnZ7bozd_6lFi29ilxvrZQIKR23iA-YKE-qmLtf16kswYQKbbZJeVTkGt8qHzdGU0NCjbG0OGh0Ma0rrLsvbBJJMM22Cs0kW5MnWwBfBrVuy2cii3PlHrfuvYQ_ybL5Rv5u1bSwdzJQh205GKKSZXQjCBCY_MI4SnysgMfKmVuK1FS3NYeIS6ypMT8zSMAWvbVMUDz8aYhdKGZK6qkGSWC09zEKw&state=12345&session_state=a9389533-fc62-4da3-8377-a697f0ad95f2#
```

> [!NOTE]  
> The important part of this is the `code` parameter.

![code highlighted](/assets/project-online/Pasted%20image%2020231102103316.png)

Now that we have the code we can get a delegated app authentication session to MSGraph

3. Make a HTTP to AAD to get an access token.

The request should be sent to the following URL

`https://login.microsoftonline.com/936f0468-e5bb-4846-b365-5ae3790deadf/oauth2/v2.0/token`

The GUID in the URL is the tenant ID

Sample parameters

| Key           | Value                                                                                                                                                                                                                                                                                |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| client_id     | afdbe70d-fbb8-47fb-9348-ad43fed0cbda                                                                                                                                                                                                                                                 |
| grant_type    | authorization_code                                                                                                                                                                                                                                                                   |
| redirect_uri  | https://oauth.pstmn.io/v1/browser-callback                                                                                                                                                                                                                                           |
| client_secret | CoZ8Q~1lYpC                                                                                                                                                                                                                                             |
| scope         | profile openid email https://graph.microsoft.com/EnterpriseResource.Read https://graph.microsoft.com/Project.Read https://graph.microsoft.com/ProjectWebApp.FullControl https://graph.microsoft.com/ProjectWebAppReporting.Read https://graph.microsoft.com/User.Read offline_access |
| code          | 0.AXwAaARvk7vlRkizZVrwvd7BlMjZTXrMxUYnl11Q-mrzATMVk_BHIPT1V5-nRKPksBFIc                                                                                                                                                                                                              |

Postman request and response

![Postman request and response](/assets/project-online/Pasted%20image%2020231102105104.png)

4. The final step is to swap this Graph access token for an SPO token. We can do this using the refresh token that was returned in the above request.

The request should be sent to the same URL as before

`https://login.microsoftonline.com/936f0468-e5bb-4846-b365-5ae3790deadf/oauth2/v2.0/token`

The GUID in the URL is the tenant ID

Parameters are slightly different


| Key           | Value                                         |
| ------------- | --------------------------------------------- |
| client_id     | afdbe70d-fbb8-47fb-9348-ad43fed0cbda          |
| grant_type    | refresh_token                                 |
| refresh_token | 0.AXwAaARvk7vlRkizZVrjeQ3q3w3n26-4-.....      |
| client_secret | CoZ8Q~-f     |
| scope         | https://m365x82565687.sharepoint.com/.default |

Postman request and response

![Postman request and response](/assets/project-online/Pasted%20image%2020231102105738.png)


## Project Rest APIs

It's now possible to call the Project Online rest APIs using the access token returned in the last postman request.

To test this in Postman, make a get request to the following URL - Update the first section to be the URL of the ProjectOnline Site Collection

`https://m365x82565687.sharepoint.com/sites/classicProject/_api/projectdata/Projects`

> [!NOTE]  
> Ensure you add the `access_token` to the Authorization tab

![Project Online data](/assets/project-online/Pasted%20image%2020231102115757.png)

## Moving to Prod

Our next task is to check if we can keep a authentication session active.

From reading documentation it apepars we can.

[access token / refresh token with MSAL - Stack Overflow](https://stackoverflow.com/questions/51332122/access-token-refresh-token-with-msal)

It looks like the refresh token is active for 90 days - So as long as the app does something every 90 days the token should be refreshed and we don't need to do anything

This is confirmed here, as long as the app is in daily use the refresh token will last forever. Cool!

[How long do refresh tokens last for? - Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/965555/how-long-do-refresh-tokens-last-for)

[Configurable token lifetimes | Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity-platform/configurable-token-lifetimes#refresh-and-session-token-lifetime-policy-properties)


## 6 days later..

Testing seems to suggest this is the case

A six day old refresh token can be used to get a new session.

Which can then be used to call the project APIs

![[Pasted image 20231108230624.png]]

So if we can Store the refreah token (securely) we can simply use this everytime to get an auth token and then retreive the data. 
