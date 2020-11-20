---
title:  "MS Flow and Parsing the response of HTTP SharePoint Request"
categories:
  - SharePoint
  - M365
tags:
  - Power Automate
  - SharePoint Rest API
---

## Welcome

What better topic to use for my first post than what I was working on the afternoon I decided to start writing this blog.

I've been meaning to look at Microsoft Flow for some time now as I keep seeing and hearing references to it. People often say why don't we use flow for that... But no one, not at least in my circles, has actually done anything noteworthy with it.

A bit of back story... I've started working part time on a new project and wanted to keep track of actual time spent working. For this I needed a quick and simple way to track the time I start and finish working. Of course, I'm sure there lots of existing apps/tools that I could use for this but where is the fun in that.

## Create the Trigger

All flows start from a trigger. As I will be deciding when I start or finish working I've opted for the Flow Mobile Button to be my trigger.

Start from a blank flow and choose Flow Mobile Button as your trigger:

![Starter for 10](/assets/ms-flow/Flow-Button.png)

Being a SharePoint guy, I thought the best place to store this data will be in a SharePoint List. So I created a list called TimeSheets with the following columns.

**Display Name**|**Name**|**Type**|**Description**
:-----:|:-----:|:-----:|:-----:|:-----:
Title|Title|string|Item title
Start / End|Start_x0020\__x002f\__x0020_End|Choice|Start or End record
Time|Time|Date and Time|Time started or finished work
Resource|Resource|Person or Group|The user that this record is about

Of course if you create your list from code you would not end up with that ridiculous internal name for Start / End. An oversight on my part there...

## Add the action

Next step was to add the action. Again Flow as loads of options here, Create SharePoint Item being the obvious choice. However, let's say we wanted to do something that didn't have a pre-existing action. The Send a HTTP request to SharePoint sounded like it would do the trick. 

We will be using the endpoint `/_api/web/lists/getbytitle('TimeSheets')/items` where `TimeSheets` is the name of your list. This endpoint will except a `POST` request with the details of our new list item.

Composing the HTTP request was a bit of trial an error. Compose the flow and test it. I was pleasantly surprised with the response, some actually useful error information! The first error I came across was: 

`The property '__metadata' does not exist on type 'SP.Data.TimeSheetsListItem'. Make sure to only use property names that are defined by the type`

Looking at the [API reference](https://msdn.microsoft.com/en-us/library/office/dn531433.aspx#bk_ListItem) for the request showed me, I was missing some header information. 

![HTTP SharePoint Request Broken](/assets/ms-flow/HTTPRequest.png)

My next error was not so simple:

`A 'PrimitiveValue' node with non-null value was found when trying to read the value of a navigation property; however, a 'StartArray' node, a 'StartObject' node, or a 'PrimitiveValue' node with null value was expected`

 I guess this is down to me trying to enter a string into a User Group column... that's not going to work. I came across [this StackExchange](https://sharepoint.stackexchange.com/questions/71827/rest-post-how-to-add-a-list-item-with-people-and-group-choice-and-url-f) question that made me realise I could populate the `ResourceId` instead of the `Resource` field. With a quick lookup and bit of hard coding my new request worked!

![HTTP SharePoint Request Working](/assets/ms-flow/HTTPRequestWorking.png)

However, we want this Flow to work for others not just one person. We need to make this dynamic... Flow dynamic content gives you access to UserName and UserEmail. Time for another action.

## Add a pre action

Again we will be using a HTTP SharePoint Request. This has become just as much a REST exercise as a Flow, but that's sort of the point.

We will use the `/_api/web/ensureuser` endpoint to get user information as according to the [Microsoft API Reference](https://msdn.microsoft.com/en-us/library/office/dn499819.aspx#bk_WebEnsureUser) it will add the user to the site if they have not beed added yet. That sounds great.

You will notice from the API that body must contain LoginName for the user including claim information.

```json
{ 
    "logonName": "i:0#.f|membership|user@domain.onmicrosoft.com" 
}
```

Thankfully we can hard code the claim information and use dynamic content for the User email.

![HTTP Pre Request](/assets/ms-flow/PreHTTPReq.png)

Now we need to parse the response from the `ensureuser` request and use it in our original request. Come on Flow, don't fail us now.

## Parse JSON Action

There is a great [blog post](http://johnliu.net/blog/2018/6/a-thesis-on-the-parse-json-action-in-microsoft-flow) on the parse JSON Flow action. So I won't go into too much detail.

Add the Parse JSON action between the two HTTP requests. You want to set the Content field to the body of the 1st HTTP request. Dynamic content makes this fairly straightforward. Drop the following JSON snippet into the sample payload section to generate the schema.

```json
{
  "d": {
    "Id": 6
  }
}
```

The ensure user request returns much more data however we are only interested in the Id property so we don't need to bother about the rest.

![Parse JSON Action](/assets/ms-flow/ParseJSON.png)

## Include parsed fields in SharePoint HTTP Request

Final step is to include the parse field in you original (now second) HTTP Request. The new fields will appear in the dynamic content window under a Parse JSON heading. You may need to click the *See more* button to have the list display the `Id` field we are after. 

![Update HTTP Request](/assets/ms-flow/UpdateHTTPReq.png)

## Run that flow

Hopefully everything will now execute and create a record in our list as intended. You should end up with something similar to the below.

![Final Flow](/assets/ms-flow/FinalFlow.png)

Now go and check your list

## Final thoughts

At the moment we can only have functionality to start working. But now that the hard part is done, I'm sure you can figure out how to add some inputs to your flow button and use that input to send a slightly different request. Let me know how you get on.

Being able to chain up these sorts of HTTP Requests means that anything you can achieve in SharePoint using REST you can now do using Flow. Pretty powerful stuff. I know we completed a fairly simple task today but with some thought and a massive flow I'm sure great things can be achieved. 
