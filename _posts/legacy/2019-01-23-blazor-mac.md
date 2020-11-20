---
title:  "Blazor on Visual Studio for Mac"
categories:
  - Visual Studio for Mac
  - Web dev
tags:
  - Blazor
  - C#
  - dotnet
---

Whilst looking into something else (SignalR), I came across Blazor. A cool new web-dev framework for dotnet and C# developers.  

I mean I do try my best with JavaScrip and React. I would even go as far as saying that I'm getting quite good but I still feel much more at home when I'm using dotnet and writing C#

So of course I needed to get this Blazor stuff up and running.  

## To the Docs

The [docs](https://blazor.net/docs/get-started.html) tell you that you need Visual Studio 2017 and the latest Blazor Language extension. But of course being one on the VS for Mac users I don't have either of those.

Thankfully we have the command line tools. To install the blazor templates use:

    dotnet new -i Microsoft.AspNetCore.Blazor.Templates::*

This will install four blazor templates for you. You can check using:

    dotnet new -l

This will list your available templates.

![Blazor Templates](/assets/blazor/blazor-templates.png)

Now that you have your templates you can create a blazor project and run it.

    dotnet new blazor -o BlazorClient
    cd BlazorClient
    dotnet run

You should now see your Blazor App running on ports 5000 and 5001.

![Blazor Running](/assets/blazor/blazor-running.png)

Browse to http://localhost:5000, you should see a nice Blazor web app.

![Blazor Web](/assets/blazor/blazor-web.png)

You can open your project in Visual Studio for Mac and get coding ;)

## Troubleshooting

If like me, when you ran `dotnet run` things didn't quite work out and you get a build error like the below:

    /Users/alexgover/.nuget/packages/microsoft.aspnetcore.blazor.build/0.7.0/targets/Blazor.MonoRuntime.targets(633,5): error MSB3073: The command "dotnet "/Users/alexgover/.nuget/packages/microsoft.aspnetcore.blazor.build/0.7.0/targets/../tools/Microsoft.AspNetCore.Blazor.Build.dll" write-boot-json "obj/Debug/netstandard2.0/BlazorClient.dll" --references "/Users/alexgover/Projects/BlazorClient/obj/Debug/netstandard2.0/blazor/bootjson-references.txt" --embedded-resources "/Users/alexgover/Projects/BlazorClient/obj/Debug/netstandard2.0/blazor/embedded.resources.txt" --linker-enabled --output "/Users/alexgover/Projects/BlazorClient/obj/Debug/netstandard2.0/blazor/blazor.boot.json"" exited with code 150. [/Users/alexgover/Projects/BlazorClient/BlazorClient.csproj]

    The build failed. Please fix the build errors and run again.

Then it's likely you will need to update your version of dotnet core.

To find out your version of dotnet core use:

    dotnet --version

When mine was failing I was using version 2.1. Upgrading this to version 2.2 sorted out all my problems.

You can download dotnet core version 2.2 [here](https://dotnet.microsoft.com/download?initial-os=macos)
