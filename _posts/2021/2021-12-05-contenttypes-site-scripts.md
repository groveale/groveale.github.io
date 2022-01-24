---
title:  "Global Content Types and Site Scripts"
categories:
  - SharePoint
  - M365
tags:
  - Site Scripts
  - Content Types
---

I've been my OG solution to make use of global content types in new sites for some time now. The solution involves

* A site script
  * Triggered on new Team site creations, simply invokes a flow.
* A flow trigger
  * Adds a URL to the storage queue
* An azure storage queue
  * A list of URLs
* An azure function
  * Runs on a cron, picks up the URLs from the queue, checks if the content types are available in the site and executes a site scripts if so, add the URL back onto the queue if not.
* Another site script
  * Adds the content types to the default document library so they are surfaced in teams

![OG-solutions](/assets/site-scripts/content-type-og-solution.png)

The main issue this solution resolved was that we never knew how long the content types would take to pull down from the content type hub. Could be anywhere between 1 hour to 24 hours. Then Microsoft moved all the flow triggers and connectors I was using into the premium tier... but we won't dwell on that.

A couple of months ago (September 2021) Microsoft finally updated the content type hub, giving it a nice modern look and with it (and more importantly) some CSOM endpoints that enable us to pull these content types ourselves.

```c#
List<string> ContentTypes = new List<string>({"0x0101", "0x01" })

var sub = new Microsoft.SharePoint.Client.Taxonomy.ContentTypeSync.ContentTypeSubscriber(ClientContext);
ClientContext.Load(sub);
ClientContext.ExecuteQueryRetry();

var res = sub.SyncContentTypesFromHubSite2(ClientContext.Url, ContentTypes);
ClientContext.ExecuteQueryRetry();
```

and of course a PnP PowerShell cmdlet

```powershell
 Add-PnPContentTypesFromContentTypeHub -ContentTypes "0x0101", "0x01" 
```

Great, I can now rationalise my solution. The only real change is with the Azure function. Rather that checking for the content types, it can now pull them down.

![sync-solution](/assets/site-scripts/content-type-og-solution-v2.png)

Sites are now ready in minutes rather than hours.
