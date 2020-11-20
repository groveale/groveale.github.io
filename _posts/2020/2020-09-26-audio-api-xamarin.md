---
title:  "Using the Audio APIs in Xamarin macOS"
categories:
  - Xamarin
  - Xamarin.Mac
tags:
  - macOS
  - Swift
  - C#
---

**UPDATE: I built the app! I had to use SwiftUI in the end, which was actually a great experience. Here is a [link](https://apps.apple.com/gb/app/earbuds-audio-link/id1537458726?mt=12) to the Mac App store if you want to check it out.**

Over the past few months I've had a couple of use cases where I've needed to output audio to more than one bluetooth device on my mac. 

Once you have your headphone connected, it's actually quite straightforward to set up and configure using an inbuilt macos app called `Audio MIDI Setup`

![Audio MIDI Setup](/assets/mac-audio/audio-midi.png)

However there must be a better way to do this. Perhaps there are some APIs. Maybe I could even develop an app to make this even easier.

I broke this task down into three steps:

1. List available bluetooth devices and establish connections
2. List available audio outputs
3. Create a multi output device with and set it as primary output

## List Bluetooth Devices

I spent much longer on this than I should of. There is an API called [CoreBluetooth](https://docs.microsoft.com/en-us/dotnet/api/corebluetooth?view=xamarin-mac-sdk-14) which sounded like it should do what I needed. There is even a great [sample app](https://docs.microsoft.com/en-us/samples/xamarin/mac-samples/heartratemonitor/) from the Xamarin guys. However this API is only for Bluetooth low energy devices. Not traditional bluetooth devices like headphones. 

There is another framework called EAAccessoryManager

Just as i was about to give up I came across the [`IOBluetooth` framework](https://developer.apple.com/documentation/iobluetooth). This framework enables you (among other things) to scan for bluetooth devices that are in range. Perfect, this is exactly what I needed. Unfortunately there is no Xamarin bindings for the IOBluetooth APIs, this make things a little tricky to test. Before I went any further (creating my own [bindings](https://docs.microsoft.com/en-us/xamarin/mac/platform/binding)) I needed to confirm that the `IOBluetooth` framework does what I need. So I knocked up a quick xcode playground in swift. Thanks to Dylan for his answer to this [SO question](https://stackoverflow.com/questions/40636726/nearby-bluetooth-devices-using-swift-3-0?noredirect=1&lq=1). Most of the code is from there, I just added the parring code.

```swift
import Cocoa
import PlaygroundSupport
import IOBluetooth

class BlueDelegate : IOBluetoothDeviceInquiryDelegate {
    func deviceInquiryStarted(_ sender: IOBluetoothDeviceInquiry) {
        print("Inquiry Started...")
    }
    func deviceInquiryDeviceFound(_ sender: IOBluetoothDeviceInquiry, device: IOBluetoothDevice) {
        print("\(device.addressString!)")
        print("\(device.name!)")
        // Now we have a device let's pair it
        let pariingAttempt = IOBluetoothDevicePair.init(device: device).start()

    }
    func deviceInquiryComplete(_ sender: IOBluetoothDeviceInquiry!, error: IOReturn, aborted: Bool) {
        print("Inquiry Complete...")
        
    }
}
var delegate = BlueDelegate()
var ibdi = IOBluetoothDeviceInquiry(delegate: delegate)
ibdi?.updateNewDeviceNames = true
ibdi?.start()
PlaygroundPage.current.needsIndefiniteExecution = true
```

Testing this code with my Bluetooth speaker confirms that it works!

```bash
Inquiry Started...
00-12-6f-b6-cf-30
Als B1
Inquiry Complete...
```

Now that we can list nearby available devices and establish a connection we can move onto the next task.

## List Available Outputs

At first this looked like it was going to be a very simple task. The [`AVFoundation` API](https://docs.microsoft.com/en-us/dotnet/api/avfoundation?view=xamarin-mac-sdk-14) has a great little property `AVCaptureDevice.Devices`. This lists all available devices for capturing audio / video.  

```csharp
var inputDevices = AVCaptureDevice.Devices;

foreach (var device in inputDevices)
{
    Console.WriteLine($"{device.LocalizedName}, {device.UniqueID}");
}
```

When I run this I get the following, the built in mic and Camera on my mac.

```bash
Built-in Microphone, AppleHDAEngineInput:1F,3,0,1,0:1
FaceTime HD Camera (Built-in), 0x1410000005ac8600
```

So now I just need the opposite of this, list all devices for audio output. Not quite as simple...

We could use the IOBluetooth API again but this would only work for Bluetooth devices, we need to list all the audio output capable devices however they are connected.

I could not find anything in the `AVFoundation` API that would help me so i started looking into another API, [`ExternalAccessory`](https://docs.microsoft.com/en-us/dotnet/api/externalaccessory?view=xamarin-mac-sdk-14). At first this looked promoising, the docs describe the `ExternalAccessory` namespace as classes for communicating with accessories connected to the device. Sounds ideal, I knocked up the below code hoping to print a list of all my devices

```csharp
EAAccessoryManager mgr = EAAccessoryManager.SharedAccessoryManager;

var accessories = mgr.ConnectedAccessories;
foreach (var accessory in accessories)
{
    Console.WriteLine(accessory.Name);
}
```

My accessories array is always empty... Upon further investigation, the `ExternalAccessory` API is intended to be used by device manufactures who enrol in the [MFi program](https://developer.apple.com/programs/mfi/). For it to work you need to add specific hardware protocol to your `Info.plist` file.

```xml
<key>UISupportedExternalAccessoryProtocols</key>
<array>
    <string>hardware.specific.protocol.from.manufacturer.goes.here</string>
</array>
```

This wasn't going to work. I needed something else, I knew there must be another way because the Spotify App has the ability to select an output device and there is no way they have all the hardware protocols for all headphones.

### AUAudioUnit

This is an API that provides low level audio access. Perhaps this would work. I found some [swift code](https://stackoverflow.com/questions/51934514/list-all-available-audio-devices/58618034#58618034) that does exactly what i need. So I loaded it up in xcode playground and gave ran it.

```bash
Found device "Built-in Output", uid=AppleHDAEngineOutput:1F,3,0,1,1:0
Found device "Multi HeadPhones", uid=~:AMS2_StackedOutput:0
```

Nice, now to find the `AudioObjectGetPropertyData` method in Xamarin... There's a discussion [here](https://forums.xamarin.com/discussion/61196/audioobjectgetpropertydata-not-found-in-xamarin) about there whereabouts of this exact method. To summarise, it's not there but Damian has shared a solution to expose the native APIs. Nice work Damian.

```csharp
[DllImport(ObjCRuntime.Constants.AudioUnitLibrary)]
static extern uint AudioObjectGetPropertyData(
    uint inObjectID,
    ref AudioObjectPropertyAddress inAddress,
    ref uint inQualifierDataSize,
    ref IntPtr inQualifierData,
    ref uint ioDataSize,
    out uint outData
);
```

Now I know how to expose the method I need to request the right data. As i want to create a multi output device I will likely need the device identifiers, names would be good too.

Unfortunately the [`AudioUnit.AudioObjectPropertySelector`](https://docs.microsoft.com/en-us/dotnet/api/audiounit.audioobjectpropertyselector?view=xamarin-ios-sdk-12) enum doesn't have the full list of properties, I found the ones I needed and printed the values in xcode playground. I then created two static variables in my class so i was able to use them in my function.

```csharp
public static uint kAudioDevicePropertyDeviceUID = 1969841184;
public static uint kAudioDevicePropertyDeviceNameCFString = 1819173229;
```

Now I needed to figure out how I could output a `string` rather than a `uint`. My first approach was to attempt to replicate the swift code and create another `AudioObjectGetPropertyData` function but this time with a `CFString` out parameter rather than `uint`. 

However my code exceptioned:

`Type CoreFoundation.NativeObject which is passed to unmanned code must have a StructLayout attribute.`

I had another look at the [Apple docs](https://developer.apple.com/documentation/coreaudio/1422524-audioobjectgetpropertydata?language=objc) for the `AudioObjectGetPropertyData` function. How could I have an out parameter of the equivalent of swift `void`. I noticed in Damian's code that the `inQualifierData` had a type of `IntPtr` and in the apple docs this variable has a type of `void`. I looked up `IntPtr`, turns out it's a `Struct` - That corresponds with the earlier error. Let's give it a go. 

```csharp
[DllImport(ObjCRuntime.Constants.AudioUnitLibrary)]
static extern uint AudioObjectGetPropertyData(
    uint inObjectID,
    ref AudioObjectPropertyAddress inAddress,
    ref uint inQualifierDataSize,
    ref IntPtr inQualifierData,
    ref uint ioDataSize,
    out IntPtr outData
);
```

I am now able to use the `CFString(outData)` constructor to get the string value of the pointer. Boom! we have our audio device information.

`Multi HeadPhones, uid=~:AMS2_StackedOutput:0`

The full class can be found at this [gist](https://gist.github.com/groveale/679ff81ce4c0fba03751c204f74fde21)

I can now move onto the last part of the puzzle. Creating a multi output device.

## Multi Output Device

Last but not least I now need to create the Multi Output Device and add all the devices to it and finally set it as the primary output for the mac.

This [SO question](https://stackoverflow.com/questions/35469569/how-can-i-programmatically-create-a-multi-output-device-in-os-x) has an answer that has provided what we need.  The [`AudioHardwareCreateAggregateDevice`](https://developer.apple.com/documentation/coreaudio/1422096-audiohardwarecreateaggregatedevi?language=objc) function in the `CoreAudio` framework.

Again, there does not seem to be a Xamarin binding for this function. Looks like we will need to create our own again.

![Eight hours later](/assets/mac-audio/eightHoursLater.jpg)

After a lot of trial and error I wasn't able to get the `AudioHardwareCreateAggregateDevice` method working in `Xamarin.Mac`. I ended up posting a [question on SO](https://stackoverflow.com/questions/64223652/expose-native-audiohardwarecreateaggregatedevice-method-coreaudio-xamarin-mac) detailing my approach to expose and use this function. Hopefully someone will be able to help me out and I can come back and update this post.

I'm still planning to roll this all up into an app but it's looking like I will need to use the native tools to do so. Not necessarily a bad thing, I've been looking for an excuse to turn my hand to `SwiftUI`.

## References

Swift IOBluetooth [SO question](https://stackoverflow.com/questions/40636726/nearby-bluetooth-devices-using-swift-3-0?noredirect=1&lq=1)

Xamarin bindings for IOBluetooth [Nuget packages](https://inthehand.com/2018/03/30/bluetooth-with-xamarin-mac/) - Thanks Peter Foot!

Swift code for listing audio outputs [SO answer](https://stackoverflow.com/questions/51934514/list-all-available-audio-devices/58618034#58618034) - Thanks stevex

Expose native AudioObjectGetPropertyData method [thread](https://forums.xamarin.com/discussion/61196/audioobjectgetpropertydata-not-found-in-xamarin)

Create Multi-Output Device in OS X [SO question](https://stackoverflow.com/questions/35469569/how-can-i-programmatically-create-a-multi-output-device-in-os-x)