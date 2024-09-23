---
title:  "Connect to Project Online Site Collection using an app registration - The solution"
categories:
  - SharePoint
  - Project Online
tags:
  - Authentication
---

That customer that I did all that research for on accessing `Project Online APIs` in my last post has asked me to build this into a solution. Cool!

This post is a deployment guide that can be used to help you set up and configure the required resources to get data from the `Project Online APIs` without a user (kind of)

In the previous post we used Postman to test the API calls, today we use PowerShell and the web browser

## App Registration (Authentication)

Create an App registration in AAD with the following permissions.

This will allow our app to sign in on behalf of our user and pull data from `projectonline`. 

![app registration permissions](/assets/project-online/Pasted%20image%2020240226215714.png)

> [!NOTE]  
> This user account needs to have read access to all projects in the Project Online Site Collection 

To support with this guide the `clientId` for this app is `cd85557e-65a9-4854-b879-2671dfaee51a`

The `tenantId` is `75e67881-b174-484b-9d30-c581c7ebc177`

You should also create a secret for the app registration - You will need this later

```
eDs8Q~k7XHsc..
```

The app requires a redirect URI - This URI will be where the code is returned that is required to authorize the on behalf of (delegated) application connection

### Testing the authentication

For testing purposes the below URI can be used - This will enable the auth code to be returned to a browser

```
https://oauth.pstmn.io/v1/browser-callback
```

![call back URI](/assets/project-online/Pasted%20image%2020231102102303.png)

Execute the following HTTP request in a browser (if authenticated you see no prompt). This browser session needs to be authenticated by the user that has access to the Project Online Site Collection.

```
https://login.microsoftonline.com/75e67881-b174-484b-9d30-c581c7ebc177/oauth2/v2.0/authorize
  ?client_id=cd85557e-65a9-4854-b879-2671dfaee51a
  &response_type=code
  &redirect_uri=https://oauth.pstmn.io/v1/browser-callback 
  &response_mode=query&scope=profile openid email https://graph.microsoft.com/EnterpriseResource.Read https://graph.microsoft.com/Project.Read https://graph.microsoft.com/ProjectWebAppReporting.Read https://graph.microsoft.com/User.Read offline_access&state=12345
```

A authentication success would be indicated as below

![Auth success](/assets/project-online/Pasted%20image%2020240226220806.png)

In the URL you will have something similar to the below:

```
https://oauth.pstmn.io/v1/browser-callback?code=0.AXwAaARvk7vlRkizZVrjeQ3q3w3n26-4-_tHk0itQ_7Qy9q7AKM.AgABAAIAAAAtyolDObpQQ5VtlI4uGjEPAgDs_wUA9P8lE2r45aczwmR0G3_abfM75HIR5yK_cPHIQz1NnUxaawKCcu8mw4jrFmNllDCyzxx5CKLhdDh-vnAoDNR0dq34_tLY96jtPevLvSH1dOorDLNqOjaZi7k2_mrnsFxADsd2ExkSQrJ3PxUjUnPCChn52r10YXJ9P_GP6PmlI_fkQfNNovC2yQMw5OO9bkYVmnUfiRU0Hhq4LmPSVcH9oTSrWaEsSC9js4ZLpMUIbolo_EaXIKfxEprpeJZ0tXKbqJizQqqRjnqOcDBRWMpBS-xBHPgSovV5bchlultczfu5A107-d0sfLkUyOe7tqahXJFOKrTFKG2IIzCFB2OfPNp0qbc42aEq2PUw6wz7kSVgXWYAg3hX3Jo3HQi7_3bsK7aU_q1SWhB-59Sevh3dfcFB4rvLKAEP13fi5H71G-5eC7X7jlat9ix6fQ2qzXc0aA3NopqnZ7bozd_6lFi29ilxvrZQIKR23iA-YKE-qmLtf16kswYQKbbZJeVTkGt8qHzdGU0NCjbG0OGh0Ma0rrLsvbBJJMM22Cs0kW5MnWwBfBrVuy2cii3PlHrfuvYQ_ybL5Rv5u1bSwdzJQh205GKKSZXQjCBCY_MI4SnysgMfKmVuK1FS3NYeIS6ypMT8zSMAWvbVMUDz8aYhdKGZK6qkGSWC09zEKw&state=12345&session_state=a9389533-fc62-4da3-8377-a697f0ad95f2#
```


> [!NOTE]  
> The important part of this is the `code` parameter.

![Auth code](/assets/project-online/Pasted%20image%2020231102103316.png)

Now that we have the code we can get a delegated app authentication session to MSGraph

Make a HTTP to AAD to get an access token.

The request should be sent to the following URL

`https://login.microsoftonline.com/75e67881-b174-484b-9d30-c581c7ebc177/oauth2/v2.0/token`

The GUID in the URL is the tenant ID

Sample parameters

| Key           | Value                                                                                                                                                                                                                          |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| client_id     | cd85557e-65a9-4854-b879-2671dfaee51a                                                                                                                                                                                           |
| grant_type    | authorization_code                                                                                                                                                                                                             |
| redirect_uri  | https://oauth.pstmn.io/v1/browser-callback                                                                                                                                                                                     |
| client_secret | eDs8Q~k7XHscra                                                                                                                                                                                                                 |
| scope         | profile openid email https://graph.microsoft.com/EnterpriseResource.Read https://graph.microsoft.com/Project.Read https://graph.microsoft.com/ProjectWebAppReporting.Read https://graph.microsoft.com/User.Read offline_access |
| code          | 0.AXwAaARvk7vlRkizZVrwvd7BlMjZTXrMxUYnl11Q-mrzATMVk_BHIPT1V5-nRKPksBFIc                                                                                                                                                        |

```PowerShell
$tenantId = "75e67881-b174-484b-9d30-c581c7ebc177"
$url = "https://login.microsoftonline.com/$tenantId/oauth2/v2.0/token"

$body = @{
    client_id     = "cd85557e-65a9-4854-b879-2671dfaee51a"
    grant_type    = "authorization_code"
    redirect_uri  = "https://oauth.pstmn.io/v1/browser-callback"
    client_secret = "eDs8Q~k7XHscra"
    scope         = "profile openid email https://graph.microsoft.com/EnterpriseResource.Read https://graph.microsoft.com/Project.Read https://graph.microsoft.com/ProjectWebAppReporting.Read https://graph.microsoft.com/User.Read offline_access"
    code          = "0.ARoAgXjmdXSxS0idMMWBx-vBd35Vhc2pZVRIuHkmcd..."
}

$response = Invoke-RestMethod -Uri $url -Method Post -Body $body

$response
```

A response like the following indicates success

![Auth success](/assets/project-online/Pasted%20image%2020240226221928.png)

The next step is to swap this Graph access token for an SPO token. We can do this using the refresh token that was returned in the above request.

The request should be sent to the same URL as before

`https://login.microsoftonline.com/75e67881-b174-484b-9d30-c581c7ebc177/oauth2/v2.0/token

The GUID in the URL is the tenant ID

Parameters are slightly different

| Key           | Value                                         |
| ------------- | --------------------------------------------- |
| client_id     | cd85557e-65a9-4854-b879-2671dfaee51a          |
| grant_type    | refresh_token                                 |
| refresh_token | 0.AXwAaARvk7vlRkizZVrjeQ3q3w3n26-4-.....      |
| client_secret | eDs8Q~k7XHscra                                |
| scope         | https://m365x82565687.sharepoint.com/.default |


```PowerShell
$tenantId = "75e67881-b174-484b-9d30-c581c7ebc177"
$url = "https://login.microsoftonline.com/$tenantId/oauth2/v2.0/token"

$body = @{
    client_id      = "cd85557e-65a9-4854-b879-2671dfaee51a"
    grant_type     = "refresh_token"
    refresh_token  = $response.refresh_token
    client_secret  = "eDs8Q~k7XHscraR"
    scope          = "https://groverale.sharepoint.com/.default"
}

$spoResponse = Invoke-RestMethod -Uri $url -Method Post -Body $body

$spoResponse
```

![SPO token success](/assets/project-online/Pasted%20image%2020240226222959.png)

It's now possible to call the Project Online rest APIs using the access token returned in the `spoResponse`.

To test this we can make another rest call, but this time to the ProjectOnline APIs

`https://groverale.sharepoint.com/sites/pwa/_api/projectdata/Projects`

```PowerShell
$projectOnlineAPI = "https://groverale.sharepoint.com/sites/pwa/_api/projectdata/Projects"  # Replace this with your PWA endpoint

$headers = @{
    "Authorization" = "Bearer $($spoResponse.access_token)"
}

$pwaResponse = Invoke-RestMethod -Uri $projectOnlineAPI -Method Get -Headers $headers

$pwaResponse
```

A list of projects indicates success

![Project Online data](/assets/project-online/Pasted%20image%2020240226223737.png)


### Token Expiration

Refresh token is active for 90 days - So as long as the app does something every 90 days the token should be refreshed and we don't need to do anything

This is confirmed here, has long as the app is in daily use the refresh token will last forever

[Refresh tokens in the Microsoft identity platform - Microsoft identity platform | Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity-platform/refresh-tokens#token-lifetime)

### Token Revocation

Refresh token can be revoked for a number of reasons. Our refresh token has been issued to a confidential clients so the following table explains the instances when the refresh token will be revoked

| Change                                      | Confidential client token |
| ------------------------------------------- | ------------------------- |
| Password expires                            | Stays alive               |
| Password changed by user                    | Stays alive               |
| User does SSPR                              | Stays alive               |
| Admin resets password                       | Stays alive               |
| User revokes their refresh tokens           | Revoked                   |
| Admin revokes all refresh tokens for a user | Revoked                   |
| Single sign-out                             | Stays alive               |

[Refresh tokens in the Microsoft identity platform - Microsoft identity platform | Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity-platform/refresh-tokens#token-revocation)



## Deploy Azure Components

An Azure function has been developed as the vehicle to automate the process above and also send the ProjectOnline data to a SQL database.

We need the following

* Resource group
* Azure SQL db
* Azure Function, with managed identity
* KeyVault, access policies

### Resource Group

A resource group will be uses as a container for all our resources

![Resource Group](/assets/project-online/Pasted%20image%2020240227112355.png)

### Azure SQL database

You may already have an existing db but for testing purpose you may want to deploy another. I have used an existing SQL server to host my db so the resource group is different to what we created above

![SQL Config](/assets/project-online/Pasted%20image%2020240227112827.png)

#### SQL Auth

Go to Connection strings and note down the ADO.NET (SQL authentication) property.

This will be needed later

### Azure Function

Create an Azure function with the below configuration.

Notable settings.
* .NET Runtime
* Version 6 (LTS), in-process model
* Windows

![Function Config](/assets/project-online/Pasted%20image%2020240227164351.png)

If you create with the these settings you will have a storage account, app service plan and app insights created for you as well as the Function app. 

The resource group should contain similar to the below

![Azure Resoruces](/assets/project-online/Pasted%20image%2020240227164640.png)

#### Function Identity

We will give the Azure function an AAD assigned managed identity. With this identity, the function can access the KeyVault to obtain the refresh token without the need for an additional app registration

![Function Identity](/assets/project-online/Pasted%20image%2020240227165341.png)

#### Function code deployment

The function code can be found here [groveale/project-online-api (github.com)](https://github.com/groveale/project-online-api)

It includes two functions. One is configured to be executed daily. The other is a HTTP endpoint that should be called form a PowerShell script (also in the repo) to add a refresh token to the KeyVault

There are numerous ways to deploy function code to Azure. They are detailed here - [Deployment technologies in Azure Functions | Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-functions/functions-deployment-technologies?tabs=windows)

An easy option is to use the Azure Function Extension in `vscode`. This requires the users to sign into their Microsoft account that has access to the provisioned Azure resources.

![Deploy Function Code](/assets/project-online/Pasted%20image%2020240227170050.png)

This opens a dialogue that enables you to select the Azure Function you have just created

![Select Function](/assets/project-online/Pasted%20image%2020240227170211.png)

The `vscode` extension now builds and compiles the code and uploads it to the hosted Azure Function.

Check the Azure Tab in the terminal for deployment status. My first attempt failed, but my second was successful

![Deployment Status](/assets/project-online/Pasted%20image%2020240228095556.png)

For a successful deployment the output tab on the terminal will contain details of the URLs for the functions. 

![Deployment output](/assets/project-online/Pasted%20image%2020240228095716.png)

FYI the `GetSecretDetails` and `UpdateRefreshToken` functions are disabled. They are still in the repo for completeness and can be used as a reference resource

![Functions Disabled](/assets/project-online/Pasted%20image%2020240228095848.png)


### Azure Key Vault

Create a KeyVault in azure with the following config

![Key Vault Config](/assets/project-online/Pasted%20image%2020240228102023.png)

Once provisioned, go to the keyvault and add role based access for our Azure Function

![add role](/assets/project-online/Pasted%20image%2020240228102430.png)


Choose `Key Vault Secrets Officer` from the list of roles. The select the managed identity option. Click Select Members and find the Azure function. Add the role

![add role to function](/assets/project-online/Pasted%20image%2020240228103011.png)

The Azure function now has permission to create / update and retrieve secrets in the KV

## Configure Azure Components

#### App Registration

Now that they Azure function has been deployed we need to add another redirect URI

This is the URL of the `UpdateRefreshTokenFromAccessCode` function.

```
https://syncprojectonlinespodataag.azurewebsites.net/api/UpdateRefreshTokenFromAccessCode
```
![Update redirect URI](/assets/project-online/Pasted%20image%2020240228141613.png)

#### Function Config

The function app contains many config variables. These can either be entered into the portal or added via the `vscode` extension

```json
{
	"clientId": "cd85557e-65a9-4854-b879-2671dfaee51a",
	"clientSecret": "eDs8Q~k7XHscraRjim...",
	"scope": "https://groverale.sharepoint.com/.default",
	"projectOnlineSiteUrl": "https://groverale.sharepoint.com/sites/pwa",
	"tenantId": "75e67881-b174-484b-9d30-c581c7ebc177",
	"fullPull": "false",
	"keyVaultName": "spo-projectonline-kvag",
	"redirectUri": "https://syncprojectonlinespodataag.azurewebsites.net/api/UpdateRefreshTokenFromAccessCode",
	"sqlConnectionString": "Server=tcp:groveale-sql-server.database.windows.net,143..;"
}
```

| Field Name           | Value                                                                                     | Description                           |
| -------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------- |
| clientId             | cd85557e-65a9-4854-b879-2671dfaee51a                                                      | AAD App Id                            |
| clientSecret         | eDs8Q~k7XHscraRjim...                                                                     | Secret for AAD App created earlier    |
| scope                | https://groverale.sharepoint.com/.default                                                 | Scope of access requested             |
| projectOnlineSiteUrl | https://groverale.sharepoint.com/sites/pwa                                                | URL of the Project Online site        |
| tenantId             | 75e67881-b174-484b-9d30-c581c7ebc177                                                      | Unique identifier for the tenant      |
| fullPull             | false                                                                                     | full pull or delta (last 24 hours)    |
| keyVaultName         | spo-projectonline-kvag                                                                    | Name of the Azure Key Vault           |
| redirectUri          | https://syncprojectonlinespodataag.azurewebsites.net/api/UpdateRefreshTokenFromAccessCode | URI to redirect after authentication. |
| sqlConnectionString  | Server=tcp:groveale-sql-server.database.windows.net,143..;                                | Connection string for SQL Server      |

Open up the extension, find the function in your list or resource. Expand and right click the app settings. Clicking upload local settings will upload the settings values from the `local.settings.json` files

> [!NOTE]  
> The repo includes a `local.settings.json.sample` file. Use this to create a local settings file for your environment

![add settings](/assets/project-online/Pasted%20image%2020240228135347.png)

The Azure function is now configured.

#### Key Vault Initial Config

There is a once time action required to seed the KeyVault secret with a refresh token. This token will be used by the function when it attempts to pull the Project Online data.

A PowerShell script `LoginAndPostTokenToFunction` is included in the repo to support this activity.

There are three variable that should be updated in this script before executing

``` PowerShell
$client_id = "cd85557e-65a9-4854-b879-2671dfaee51a"
$tenantId = "75e67881-b174-484b-9d30-c581c7ebc177"
$redirect_uri = "https://syncprojectonlinespodataag.azurewebsites.net/api/UpdateRefreshTokenFromAccessCode"
```

Executing this script will open a browser where you should login with the user that has access to the project online data. 

A successful login attempt will add the refresh token to the KeyVault and return the token to the browser

![successful login](/assets/project-online/Pasted%20image%2020240228145518.png)


#### SQL db config

There is a number of SQL create table scripts included in the repo. Please run these to create tables in your prestaging database


## Test the Solution

Everything should now be in place for you to test the solution

Simplest way is to enter the URL of the `GetProjectOnlineData` function into your browser

![Project Online Data Summary](/assets/project-online/Pasted%20image%2020240228150408.png)

This indicates that 2 project items, 2 task items and 4 resource items have been updated in SQL.


## Production Considerations

The `GetProjectOnlineData` function should be configured as a TimerTriggered function. This way there would be to HTTP endpoint exposed to trigger a pull of the data.

The `UpdateRefreshTokenFromAccessCode` should be configured to not allow anonymous access. At this stage anyone with the URL could attempt to use it to update the secret in the Key Vault. they would not be able to obtain the refreshToken value but a successful logon attempt would overwrite the key vault value and stop the solution from working

