---
title:  "Maui development on VScode for Mac!"
categories:
  - Maui
tags:
  - VsCode
---

I'm getting back in the seat after almost 2 years out. In these two years a-lot has happened. But most importantly I know work for Microsoft and my SharePoint knowledge has come on ten fold. So expect a few little tips and tricks I've picked up over the last year or so. You can also expect some big solutions too, I've got a great item level archiving solution that I will be sharing and a Sites Dashboard so expect a blog on those soon.

But I thought i'd get back to my hobby for my first post back, mobile dev. Again, it's been a while since I've written any Xamarin, it's been so long that we don't even call it Xamarin any more. As you all probably know, we have dotnet maui. So this will be plunge into the MAUI pool.

## VS for Mac end of life

Just I was dusting off the Mac Book I see that VS for Mac is being discontinued...

[Visual Studio for Mac Retirement Announcement - Visual Studio Blog (microsoft.com)](https://devblogs.microsoft.com/visualstudio/visual-studio-for-mac-retirement-announcement/)

This shouldn't really be a problem as I haven't been using Visual Studio on windows for many years. I'm a big time VS code fan. But with Xamarin and then Maui, the tools were not available for VScode so we didn't have a choice.

This has now changed with the release of the dotnet MAUI extension for VS code 

[Announcing the .NET MAUI extension for Visual Studio Code - Visual Studio Blog (microsoft.com)](https://devblogs.microsoft.com/visualstudio/announcing-the-dotnet-maui-extension-for-visual-studio-code/)

We get IntelliSense (who need this when they have GitHub Copilot - and what. a game changer that is), local debugging on the emulators and Visual Studio look solution explorer. A nice hybrid.

If you install the [extension](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.dotnet-maui) you will get C# and the C# Dev kit. ISince installing I've followed the [create a maui app](https://learn.microsoft.com/en-us/dotnet/maui/tutorials/notes-app/) tutorial without any issue. The experience is surprisingly smooth on a Mac.

Just select your platform in the vscode bottom bar and hit debug.

![select platform](/assets/vscode-mac/debug.png)

Very quickly you will have your app running on one of those beautiful iOS simulators!

![simulator](/assets/vscode-mac/iphone.png)

## Next Steps

I'm pretty excited to get back into cross platform mobile dev. I've got tons of ideas and with the help of GitHub copilot I'm a much more efficient developer. Plus I'm writing a lot of code and building some serious solutions for my job now. Not always C# but always on the Microsoft stack.

I've found this MAUI book - [Enterprise Application Patterns Using .NET MAUI - .NET | Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/architecture/maui/), which i can to ingest over the coming week.

I love a book when you are trying to do something specific, you may remember a few years back when I found [this ebook written by Adrian Hall](https://adrianhall.github.io/develop-mobile-apps-with-csharp-and-azure/).I plan to revisit this to understand what the Azure MBaaS (Mobile Backend as a Service) offering currently looks like. Either way expect some more posts from me!