---
title:  "Ignite - The Tour London"
description: A write up of an Microsoft event I attended. Ignite The Tour London
categories:
  - Event
  - M365
tags:
  - Cognitive Services
  - Microsoft Graph
  - Teams
  - Ignite
---

I attended Ignite The Tour last week, it was the first time I have had the opportunity to go to a Microsoft conference. In general it was time well spent, I think it's good for getting information on new features and services that are either newly available or coming soon. But I did find the majority of the sessions a little high level.

If you want to get up-to-speed on a new Microsoft technology then I think it's perfect for that. In hindsight I should have gone to sessions where I knew little or nothing about the technology being discussed. I found that I already knew 90% of the new SharePoint content that was being demoed but perhaps that is because I am a bit of SharePoint nerd 8-)

That being said, for a free conference I would definitely recommend it and will be going again if I get the opportunity. 

## Summary

There were two things that kept coming up in almost all the sessions that I attended. These were:

* Microsoft Cognitive Services
* The Citizen Developer

### Microsoft Cognitive Services

Microsoft Cognitive Services can best be described as an AI API. With very little understanding of the AI or Machine Learning a developer can very easily integrate these services into their apps with a very small amount of code.

In a number of different sessions the presenter seamlessly pulled up an app, whether using PowerApps, Teams, SharePoint or an ASP website. Then showed how simple it was to integrate their chosen technology with Cognitive Services. 

Whether they were demonstrating image recognition, real time language translation, sentiment analysis or some other form of AI. They alway stressed how they were not data scientists and did not understand what the neural network or model were doing under the covers but that was the whole point. Microsoft wants us to use AI and ML in our Office 365 and business applications, but does not expect or need us to be experts in the field. 

Personally I find this very exciting, as much as I would love to go and do a masters in Artificial Intelligence I am glad that I can start using AI in my solutions with needing to be a expert!

If you want more information, the slides from a session that I felt covered Microsoft Cognitive Services well can be downloaded [here](https://aka.ms/TechCommunity/MicrosoftIgniteTour/DAT30).

### The Citizen Developer

Surprisingly, a term that I had not come across before. I thought this was a new term that describes the no code approach to app development using technologies like PowerApps, Flow and SharePoint lists. But in reality Citizen developers have been hiding themselves behind other microsoft tools such as Excel for years.

It's clear that Microsoft want the Citizen developers to migrate their speadsheets and access dbs to these newer technologies. And they are making it easier and easier. You can quickly create a PowerApp or formatted SharePoint list (not yet but you will be able to soon) from an existing excel speadsheet. 

I think there is a great opportunity here for Microsoft consultants to help their customers turn their legacy citizen developed apps into something that can live and grow in Office 365.

## Highlights

There were a few session that I attended that I thought were better than others, not just for the content but more the general message they were conveying.

* Microsoft Graph: the API for Microsoft 365
* Unleash the power of Teams with the extensibility of the platform
* Diversity in Tech: Building an Accessible Workforce

### Microsoft Graph

I wanted to go to this session because I have seen more and more examples of Microsoft graph usage in the SharePoint space. Before the session i thought Microsoft Graph was simply another API we could use to manage Office 365, similar to CSOM in SharePoint or Exchange Online PowerShell. But I realised that it's more than that, yes you can provision sites using the Microsoft Graph but it is primarily a data platform.

An interesting use case recently came up on customer site where they wanted to know if it was possible to figure out who had been involved in completing a piece or work or project which had happened a year ago. From my understanding this is the exact sort of thing you can do with the Microsoft Graph. In theory you can traverse the relationships between files, users, mail, meetings, etc and figure out everything that happened and who was involved to complete a task. This terrible photo makes it seem like it should be possible!

![Graph Relationships](/assets/ignite/graph-relationships.PNG)

I would recommend getting to grips with the Microsoft Graph, with the ever increasing cross over between Office 365 technologies I believe it will become more and more relevant. I would not be surprised if it becomes the single API for managing all of Office 365. With the release of the beta or v2 endpoint you can start to see that this is what Microsoft are trying to do.

A great place to get started with the Microsoft Graph is with the [Graph Explorer](https://aka.ms/ge). This will build the rest queries for you and handel all the authentication and even let you view the responses. You can even log in and run the queries against your own data in your tenant. I would recommend doing this, it's amazing to see how fast you can retrieve all the mail in your inbox in JSON form.

There is both a [replay (from Ignite)](https://youtu.be/ZRsrwLi-deA) and slide download [link](https://aka.ms/TechCommunity/MicrosoftIgniteTour/INT20) for this session.

### Extending Teams

Before attending this session I did not realise the maturity of the teams platform for app development.

From what I understand there are three core types (or contexts) for Teams Apps.

* Teams and Channels - Public collaboration and workflows, e.g. a service incident
* Chat - This is 1 - 1 collaboration, approvals between a manager and direct report
* Personal - User centric, aggregating content, e.g. show all tasks assigned to me

There is a powerful app called the App Studio. This is a Teams app that enables a Citizen Developer to build an app directly within Teams. Looks like really smart stuff. Having the ability to build, configure and deploy a Teams App from within Teams with no code. I'm going to need to try this out! Here are the [docs](https://docs.microsoft.com/en-us/microsoftteams/platform/get-started/get-started-app-studio) if you want to give this a go yourself.

It's also worth noting that there are two app stores within Teams. SharePoint has a similar distribution model.

* Teams App Store - This is where you can find third party apps
* Tenant App Catalogue - This is for your in house apps

The demo that was provided gave a real insight into the high level of user experience that can be achieved with a teams app. It also highlighted to me that tabs shouldn't be thought of as static, you may want a tab that deep links to a in progress item or record that is only of interest to the team for a small amount of time.

I know that Microsoft have been saying that they want Teams to be the single place that users conduct all of their day-to-day activities. With the sort of extensibility that is available I can really see it. The only issue I can see in the way is user adoption, I'm more than happy to use @ mentions to trigger workflows and use context cards to interact and perform actions but not everyone will be.

The final point mentioned was that they want to build a Teams dev / extension community. And why wouldn't they after the success of PnP in SharePoint.

Again, there is both a [replay (from Ignite)](https://youtu.be/kqekJ8MwtoY) and slide download [link](https://aka.ms/TechCommunity/MicrosoftIgniteTour/INT30) for this session.

### Diversity in Tech

This was definitely the most interesting session that I attended over the course of the two days. It was all about creating a more inclusive and accessible workforce. They had the president (Partrick Stolpmann) of the IPC (International Paralympic Committee) speaking about the success and impact that the paralympic games have had in recent years. They also had Dan Brooke from Channel 4 who was in charge of the Paralympic coverage for both London 2012 and Rio 2016. He spoke about how they did the opposite of what the BBC had done in the past. For example, their aim was to have at least 1/2 of presenters with a disability, BBC had never had anyone. They didn't play the sympathy card and try to hide but rather gave it the Nike branding style. Check out the trailer for [Rio 2016](https://www.youtube.com/watch?v=IocLkk3aYlk) and see for yourself. 

He went on to say how channel 4 looked at there other clients and set up campaigns where they offered free advertising if they used disabled actors in the adverts. They had entires from Mars, for a Maltesers advert, who weren't sure if they wanted to run them as they worried about the backlash that they might receive. Channel 4 ran the adverts anyway and it was Mars most successful advertising campaigns that they have ever conducted.

Microsoft then went on to explain about some of the great accessibility features that they have introduced:

* [Microsoft Soundscape](https://www.microsoft.com/en-us/research/product/soundscape/) - Maps delivered via sound
* [Presentation translator](https://www.microsoft.com/en-us/garage/profiles/presentation-translator/) - Display live, translated subtitles
    * A feature designed for users with hearing issues but great for international audiences
* Colour Blindness Filter - Quickly toggle a filter with Windows Key + Ctrl + C
* Magnifier - Quickly zoom in with Windows Key and + 
    * Great for Demos, I saw loads of presenters using this during the conference
* [Accessibility Checker](https://support.office.com/en-us/article/use-the-accessibility-checker-to-find-accessibility-issues-a16f6de0-2f39-4a2b-8bd8-5ad801426c7f) - Analyses your Office 365 content to check everyone can easily read and edit it
* Transcripts in Stream
    * Makes video content searchable, you would be able to quickly search all recorded meeting content and find exact points in the meeting where the item was mentioned
* [PowerPoint Designer](https://support.office.com/en-us/article/create-professional-slide-layouts-with-powerpoint-designer-53c77d7b-dc40-45c2-b684-81415eac0617) - Generates layouts and designs based on content
    * Came about from feedback from disabled users saying they could not perform the 100 plus clicks to create a good slide
    * Feature can now be used by all to achieve productivity gains.
* [Xbox Adaptive Controller Packaging](https://www.youtube.com/watch?v=8hWft3fUWTY) - Designed so that a customer who only has one hand can still open their product with ease
    * Won Awards, changing the way manufacturers package their goods for all. No one likes all that plastic anyway!

They ended the session with an announcement of [The Microsoft Accessibility Program](https://www.microsoft.com/en-us/ai/ai-for-accessibility-grants?activetab=pivot1%3aprimaryr2). To summarise there is $25 million of Azure credits that are available to businesses who want to work with charities/organisations to create solutions in the following areas:

* Employment
* Daily Life
* Communication and Connection

I attended this session on a bit of a whim, it looked like it was going to be interesting and it really was. Lesson for me here, don't just attend the tech focused sessions at these sorts of events! 

Unfortunately I can't find the slides for this session. So you will just have to take my word for it.