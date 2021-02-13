---
title:  "Use native vectors in the Xamarin.Forms Shell TabBar"
categories:
  - Xamarin
  - Xamarin.Forms
tags:
  - SVG
  - Design
---

If like me, you've been developing Xamarin.iOS and Xamarin.Android apps for some time then you will be used to implementing things in a certain way. A recent example that I came across is dealing with local image assets.

So in iOS, I like to store my images as .pdf vectors. This way I don't need to mess around with different sizes and can just have one copy of the image. I store these as `Image Sets` in [`Assests.xcassets`](https://docs.microsoft.com/en-us/xamarin/ios/app-fundamentals/images-icons/displaying-an-image?tabs=macos). Bonus here being that I get dark and light theme handling out of the box. Great!

In Android, I store my images in the `drawable` folder as .xml `SVGs`. Again, this is for convenience as I don't need to generate all the different sizes for different resolutions.

Recently I've been developing a new app using Xamarin.Forms - One of the first things I needed to do was build the TabBar, naturally I went to the [docs](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/app-fundamentals/shell/tabs#bottom-tabs).

## TabBar Icon

Let's set the tab bar icons. Looks simple enough from the docs. The `Icon`, property of the TabBar is of type `ImageSource`. 

The [docs](https://docs.microsoft.com/en-us/xamarin/xamarin-forms/user-interface/images?tabs=macos) describe four different ways that we can set an `ImageSource`

* FromFile - Requires a filename or filepath that can be resolved on each platform.
* FromUri - Requires a Uri object, eg. new Uri("http://server.com/image.jpg") .
* FromResource - Requires a resource identifier to an image file embedded in the application or .NET Standard library project, with a Build Action:EmbeddedResource.
* FromStream - Requires a stream that supplies image data.

The best option for my use case will be `FromFile`. For this to work I will need the same file path on each platform... but I want to use the native options that I know and love... Hmm.

So I just went for it. Created my assets, added the .pdf vectors to the iOS project as ImageSets in `Assets.xcassets`.

![iOS-assets](/assets/native-vectors/iOS-assets.png)

Created my .xml `SVGs` and added them to the `drawable` folder.

![android-drawables](/assets/native-vectors/android-drawables.png)

Then in my `AppShell.xml` I needed to write some `OnPlatform` code to handel the different names.

``` csharp
<TabBar>
        <Tab Title="Friends"
             Route="FriendsPage">
            <Tab.Icon>
                <OnPlatform x:TypeArguments="FileImageSource">
                    <OnPlatform.Platforms>
                        <On Platform="iOS" Value="FriendTab" />
                        <On Platform="Android" Value="tab_friends" />
                    </OnPlatform.Platforms>
                </OnPlatform>
            </Tab.Icon>
            <ShellContent ContentTemplate="{DataTemplate local:FirendsPage}" />
        </Tab>
        ...
```

Awesome. It just works, not sure why I was surprised.. I just think this should be documented, as I'm sure this is how a lot of developers would use this.

![iOS-assets](/assets/native-vectors/iOS-tabs.png)