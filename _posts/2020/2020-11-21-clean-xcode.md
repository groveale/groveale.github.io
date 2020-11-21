---
title:  "Free up some space on your Mac by cleaning Xcode"
categories:
  - Xamarin
  - Xamarin.iOS
  - Visual Studio for Mac
tags:
  - Xcode
  - Automation
  - Bash
---

If you're an Xamarin developer using `Visual Studio for Mac` building apps for `iOS` you will know that you need to have `Xcode` installed and kept up-to-date.

This means every time a new version of `iOS` is released you need to updated `Xcode` so you can compile against the latest version `iOS`. This new version comes with a new version of the SDK and a number of simulators running the latest OS.

Unfortunately for us, `Xcode` does not clean up any of it's old SDKs or simulators. It doesn't use them but for some reason keeps them around. `Xcode` is a hoarder ;)

You could argue the case for keeping SDK versions if you have some physical development devices that are not updating anymore. Bu even if you do delete these SDK versions, Xcode will always go and download these SDKs if needed.

## Delete unavailable simulators

The first and easiest step is to delete all of the unavailable simulators. The following command will list all of the simulators you have installed.

``` bash
xcrun simctl list devices
```

![list simulators](/assets/clean-xcode/list-simlualtors.png)

Notice that a number of my simulators have the message: `(unavailable, runtime profile not found)`. These are the out-of-date simulators that have quite literally no use.

To delete these run:

``` bash
xcrun simctl delete unavailable
```

I had simulators for 2 older SDK versions. Deleting these freed up almost 5GB. A big win for me, considering that's about 2% of my local disk.

## Delete old SDK versions

The next step is to delete the old iOS SDKs from the `iOS DeviceSupport` folder. These are the iOS versions from attached physical devices. As I said earlier, these are safe to delete as the folders are simply re-created when you attach your device.

`~/Library/Developer/Xcode/iOS DeviceSupport`

![old SDKs](/assets/clean-xcode/DeviceSupport.png)

Deleting these four old SDKs has just freed up another 9GB.

## Last word

With these two simple steps I have freed up a total of 14GB, about 6% of my local disk. I try to do this every now an again. Normally when I run out of storage. Perhaps this is actually a sign that I need to upgrade my Mac...
