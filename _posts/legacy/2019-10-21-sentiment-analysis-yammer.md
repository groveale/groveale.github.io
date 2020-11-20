---
title:  "Sentiment Analysis in Yammer"
description: A write up of an Microsoft Hackathon I attended including the SPFx webpart code
categories:
  - M365
  - Event
tags:
  - SPFx
  - Cognitive Services
  - Power Automate
  - Microsoft Graph
  - Teams
---

Last week I attended a great *free* [event](https://www.eventbrite.co.uk/e/global-microsoft-365-developer-bootcamp-tickets-68109556517?#) at the Microsoft Reactor London.

The event was focused around building Teams extensions with SPFx. I find events like this a great opportunity to take time off from the day to day and allocate some time to really focus on a technology. Having MVPs and Microsoft Consultants at the ready to answer your queries also really helps boost your understanding.

Anyway... in the afternoon there was a short hackathon where we needed to build a teams extension. My plan was to use cognitive services to analyse sentiment in Yammer then display this in Teams.

I thought this sort of fit the hackathon brief. Plus I could use some of those technologies I heard so much about at that Ignite even I attended a few months ago.

I broke this down into a few steps

* Yammer Post is published or a comment is added to an existing post
* MS Flow is triggered.
  * Receives post/comment data from Yammer.
  * Sends post content to Azure Cognitive services
  * Waits for Sentiment score
  * Push Sentiment score and post details to a SharePoint list for persistent storage and historic analysis
* Display in Teams

## MS Flow

As this was a hackathon I had to move quickly. First task was to get the flow working. After I found the most suitable Yammer Flow action (`When there is a new message in a group`). I had to try and figure out how to get the id of a yammer group (as the group Id field was not populating).

Turns out I only had the All Company yammer group.. which is not supported for this flow action.. So I quickly created an aptly named group and it appeared straight away.

Next was to get the actual sentiment analysis working. Using Flow this is actually very straightforward, there is a `Detect Sentiment` action which takes a string input and a language.

Note. You will need an [Azure Cognitive Services](https://azure.microsoft.com/en-gb/services/cognitive-services/) endpoint to configure the connection for flow action connection.

Finally I needed to send the data to some form of persistent storage. Of course I opted for a SharePoint list.

![Yammer Flow action](/assets/sentiment/yammer-flow.png)

## Teams and SPFx

Now I had my data I needed a way to display it in teams using SPFx. After all the main point of this day out was to get more up to speed with this technology.

I wanted something visual, the Flow is grabbing user / author information from the posts so my plan was to get the profile photo from MSGraph and then display a sentiment score for the user next to their photo. This would all happen in an SPFx webpart which could be deployed either to Teams of a modern SharePoint page.

After some hacking this is what I ended up with.

![Teams webpart](/assets/sentiment/teams_webpart.png)

The webpart uses the details of the logged in user to get the sentiment of all their posts then works out an average. It uses both the MSGraph and SharePoint Rest APIs to get the user and list item details.

I won't go into detail here but here is a [link to the repo](https://github.com/groveale/reactor-hackathon) if you want to try it out for yourself. I will write a post on some of the cool stuff the webpart does.

## Last word

At the end of the hackathon we all presented back our solutions to the group. The top three were awarded a prize, a believe it or not, mine was awarded first place. A £50 amazon voucher and a pair of Yammer Socks. A great day out of the office and I will be keeping an eye out for more events like this in the future.

Oh and did i mention it also works in SharePoint. Nice.

![SharePoint webpart](/assets/sentiment/sp_webpart.png)
