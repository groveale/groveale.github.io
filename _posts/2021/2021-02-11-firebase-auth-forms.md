---
title:  "Firebase Auth (SMS) in Xamarin Forms"
categories:
  - Xamarin
  - Xamarin.Forms
tags:
  - Firebase
  - Auth
---

Last year I updated a native app (Xamarin.iOS and Xamarin.Droid) to use Firebase Auth. The authentication for this app used SMS. My original solution was using Twilio, nothing wrong with the service from Twilio however we were looking to expand the app to multiple regions. This come with a hefty price tag, $1 per region per month. That's before we have even sent any SMS messages. Firebase offer all this for me to naturally I needed to migrate to this service.

Now I find myself in a situation where I am developing a Xamarin.Forms app which will also use SMS for authentication. So I thought why not blog along the way.

## Why Firebase Authentication

As I've already said, it's Free! Well... you get up to 20K SMS messages per month on the free plan (Spark). See [here](https://firebase.google.com/pricing?authuser=0) for pricing details.

There are also many different authentication options, simple email / password, phone (SMS) and all the federated providers you'd expect. This is wrapped up into a simple to use API that has many client SDKs. Thank fully for us Xamarin developers there are SDKs for both Xamarin.iOS and Xamarin.Droid. Whilst we will have to write code for both of these platforms, I will explain how simple this can be.

## Create Firebase App

Firstly create the Firebase project. Go to [https://console.firebase.google.com](https://console.firebase.google.com) and click on Create a project.

![Create Project](/assets/firebase-auth/create-project.png)

After you have created your projects you will need to crete both the iOS and Android versions. This is done from Project Overview tab in the console.

![Create iOS and Android](/assets/firebase-auth/create-iOS-Driod.png)

Download the google-services.json for the Android project and the GoogleService-Info.plist for the iOS project.

There is a configuration item that we need when configuring the Android application but we will come back to that later.

As instructed, add these files to the route of your Android and iOS solutions within your Forms app. Set the build action to `BundleResource` for the iOS project.

COME BACK for Android

![Build Action](/assets/firebase-auth/build-action-iOS.png)

## Configure Native Projects

In the Android project, add the NuGet packages `Xamarin.Firebase.Auth` and `Xamarin.Firebase.Core`

![Nuget Android](/assets/firebase-auth/android-packages.png)

For iOS you need, `Xamarin.Firebase.iOS.Auth`

![Nuget iOS](/assets/firebase-auth/ios-packages.png)

Firebased also need to be initialised Bin both projects. 

For Android.. Initialize Firebase authentication in MainActivity. Add the line
``` csharp
// Initialise Firebase before LoadApplication is called in OnCreate.
FirebaseApp.InitializeApp(Application.Context);

LoadApplication(new App());
```

For iOS.. In AppDelegate.cs

``` csharp
// Initialise Firebase before LoadApplication is called in FinishedLaunching.
Firebase.Core.App.Configure();

LoadApplication(new App());

return base.FinishedLaunching(app, options);
```

## Creating the Authentication Interface

