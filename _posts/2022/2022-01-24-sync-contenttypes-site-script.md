---
title:  "Sync content types using site scripts verb: addContentTypesFromHub"
categories:
  - SharePoint
  - M365
tags:
  - Site Scripts
  - Content Types
---



I've been waiting for this verb since site scripts were released way back in 2018. I've been using all sorts of solutions to get around this, my OG solution involved two site designs, a flow trigger, an azure storage queue and an azure function. 

![OG-solutions](/assets/site-scripts/content-type-og-solution.png)

The main issue this solution resolved was we never knew how long the content types would take to pull down from the content type hub. Could be anywhere between 1 hour to 24 hours. Then Microsoft moved all the flow triggers and connectors I was using into the premium tier... but we won't dwell on that.

Then last September (2021) Microsoft finally updated the content type hub, giving it a nice modern look and with it some CSOM endpoints which enabled us to pull these content types ourselves.

```c#
List<string> ContentTypes = new List<string>({"0x0101", "0x01" })

var sub = new Microsoft.SharePoint.Client.Taxonomy.ContentTypeSync.ContentTypeSubscriber(ClientContext);
ClientContext.Load(sub);
ClientContext.ExecuteQueryRetry();

var res = sub.SyncContentTypesFromHubSite2(ClientContext.Url, ContentTypes);
ClientContext.ExecuteQueryRetry();
```

and ofcourse a PnP PowerShell cmdlet

```powershell
 Add-PnPContentTypesFromContentTypeHub -ContentTypes "0x0101", "0x01" 
```

Great, I can now rationalise my solution. The only real change is with the Azure function. Rather that checking for the content types, it can now pull them down.

![sync-solution](/assets/site-scripts/content-type-og-solution-v2.png)

Sites are now ready in minutes rather than any where between 1 hour to 24 hours.

## Site Scripts

Last week, I had an absolute breakthrough.
 
I was digging around some documentation around content types, I was actually looking for what can be configured from the [M365 DSC tool](https://microsoft365dsc.com/) (turns out not much) when I found this [article](https://support.microsoft.com/en-us/office/what-s-changed-in-content-type-publishing-609399c7-5c42-4e25-aff0-b59d4aa1867f). 

It talks about what changed with content types in September 2021 but right at the bottom in mentions a new verb for site scripts (addContentTypesFromHub) due for release by the end of 2021. So I searched high and low for some documentation on this new verb. But this article is the only reference I can find…
 
So let’s just try it out. I created the below site script, just guessing at the parameters

``` json
{
    "$schema": "https://developer.microsoft.com/json-schemas/sp/site-design-script-actions.schema.json",
    "actions": [
      {
        "verb": "addContentTypesFromHub",
        "id": "0x010100BEAAB2C628BD4D20BA873BDBAC398821"
      }
    ],
    "version": 1
}
```

When applied to a site it pulls down all content types from the hub. Amazing. So as long as we keep this action at the top of a site script we shouldn’t need any flows, queues or Azure functions.

This morning I saw this little tooltip pop up

![tool-tip](/assets/site-scripts/tool-tip.png)

So I open the schema, it's been updated!! I read the schema on Friday (3 days before I wrote this post) and it was not there.

![schema-update](/assets/site-scripts/schema-update.png)

So now I can safely say this is a site script (using the verb, addContentTypesFromHub) that will pull down just the content types you specify.

``` json
{
    "$schema": "https://developer.microsoft.com/json-schemas/sp/site-design-script-actions.schema.json",
    "actions": [
      {
        "verb": "addContentTypesFromHub",
        "ids": [ "0x010100BEAAB2C628BD4D20BA873BDBAC398821" ]
      }
    ],
    "version": 1
}
```

Expect to see some updated Microsoft docs shortly, but remember you saw it here first.