---
title:  "Sync content types using site scripts verb: addContentTypesFromHub"
categories:
  - SharePoint
  - M365
tags:
  - Site Scripts
  - Content Types
---

You may have seen my last post on content types and site scripts, but it feels like I've been waiting for this verb since site scripts were released way back in 2018.
 
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