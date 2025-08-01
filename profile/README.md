# Open Media Transport

Open Media Transport (OMT) is an open-source network protocol for high performance, low latency video over a local area network (LAN).
It has been designed to support multiple high-definition A/V feeds on a standard gigabit network without any specialised hardware.

It is also completely free and open source!

## Contents

* [Design Goals](#design-goals)
* [Features](#features)
    * [Video](#video)
    * [Audio](#audio)
    * [Network](#network)
    * [Metadata](#metadata)
    * [Discovery](#discovery)
* [Basic Concepts](#basic-concepts)
    * [Send](#send)
    * [Receive](#receive)
    * [Discovery](#discovery-1)
* [Developer Guide](#developer-guide)
    * [Libraries](#libraries)
    * [Quick Examples C#](#quick-examples-c)

## Design Goals

* Completely free, open source, and royalty free under the permissive MIT license
* High throughput, allowing for multiple high-definition feeds over a standard gigabit network
* Ultra-fast, low latency codec with less than a frame of delay
* Simple and easy to implement, with cross platform code available for most platforms (Windows + MacOS + Linux)
* Optimized to run in software on commodity hardware

## Features

### Video
* 4:2:2 video codec with Alpha offers high quality, extremely low latency and reasonable bitrates over a LAN. 
A list of bandwidth requirements for common resolutions is below. To work out for different frame rates, 30fps will always be half the bandwidth of 60fps and so on.

| Resolution / Frame Rate | Quality | Bandwidth |
|-------------------------| --------| --------- |
| 2160p60 | High | 600mbps |
| 2160p60 | Medium | 400mbps |
| 2160p60 | Low | 200mbps |
| 2160p30 | High | 300mbps |
| 2160p30 | Medium | 200mbps |
| 2160p30 | Low | 100mbps |
| 1080p60 | High | 260mbps |
| 1080p60 | Medium | 200mbps |
| 1080p60 | Low | 86mbps |
| 1080p30 | High | 130mbps |
| 1080p30 | Medium | 100mbps |
| 1080p60 | Low | 43mbps |
| 720p60 | High | 136mbps |
| 720p60 | Medium | 68mbps |
| 720p60 | Low | 45mbps |
| 720p30 | High | 68mbps |
| 720p30 | Medium | 34mbps |
| 720p30 | Low | 22.5mbps |

* Existing support for codec in FFmpeg (VMX aka vMix Codec), enabling ease of adoption into existing workflows.
* Royalty and patent free
* Optimized code for both NEON and Intel SIMD instructions, allows multiple 1080p60 encode and decodes on a single CPU core.
* Accepts multiple pixel formats for both 8-bit (UYVY, YUY2, BGRA, NV12, YV12) and 16-bit (P216, PA16) workflows.

### Audio
* Uncompressed 32 bit floating point audio of up to 32 channels
* Protocol sends only non-silent audio channels to optimize bandwidth consumption

### Network
* Extremely simple TCP protocol
* TCP chosen due to highly efficient, hardware accelerated support across much all network cards and operating systems.

### Metadata
* Timestamped bi-directional side data in XML format
* Per-frame XML metadata also supported

### Discovery
* Sources published on the network using DNS Service Discovery (DNS-SD), with robust OS support on Mac, Linux and Windows.
* Optional Discovery Server where multicast is not available.

## Basic Concepts

### Send

Send is a single source that can send video, audio and metadata to multiple receivers on a network.

Send listens on a TCP port in the range 6400-6600 by default.

### Receive

Receive is a single end point that connects to a Send source to receive the video, audio and metadata from that source.

### Discovery

Discovery is how a **Send** publishes its name on the network so any Receive can connect to it.

Discovery can use either DNS-SD are multicast UDP discovery protocol, or a dedicated Discovery Server using TCP where multicast is not possible.

Send names are in the format **DEVICENAME (Source Name)** where DEVICENAME is typically either the name of the computer (for computer sources) or a unique device name (in the case of hardware encoders).

## Developer Guide

### Libraries

Open Media Transport provides two cross-platform libraries:

**libomtnet and libomt**

This is the library that implements most of the functionality of Open Media Transport such as creating senders and receivers and publishing sources on the network for discovery.

libomtnet is available for .NET applications (both .NET Framework 4+ and .NET 8.0+ apps)

libomt is available for any applications that can use C-style exports (including C/C++)

Getting started guides for each are available in the respective repositories on Github.

**libvmx**

This is the video compression library that encodes/decodes video in VMX format.
This the key to enabling multiple high definition video feeds over an ordinary gigabit network.

A more in depth overview of its features and functionality can be found in the libvmx repository on Github.

### Quick Examples (C#)

Add a reference to libomtnet.dll to your project and make sure libvmx.dll (Windows) libvmx.dylib (MacOS) or libvmx.so (Linux) is in the same directory as the application.

**Sending Video**

Create a new OMTSend object, specifying the name and optionally the video quality (which in most cases should be left as Default).

```
OMTSend mySender = new OMTSend("My Sender", OMTQuality.Default);
```

Create a new OMTMediaFrame object, filling out the fields to define the type of uncompressed video you wish to send.

```
OMTMediaFrame myFrame = new OMTMediaFrame();
myFrame.Type = OMTFrameType.Video;
myFrame.Codec = (int)OMTCodec.UYVY;
myFrame.Width = 1920;
myFrame.Height = 1080;
myFrame.FrameRate = 60;
myFrame.Timestamp = -1; //Important: This makes sure we only send at a maximum of 60fps (The framerate we set for this example)
myFrame.AspectRatio = 16.0F / 9.0F;
myFrame.Stride = myFrame.Width * 2;
myFrame.ColorSpace = OMTColorSpace.BT709;
```

Fill in the frame data with your uncompressed video frame. In this example we are loading a UYVY frame from file.

```
byte[] myUYVYFrame = File.ReadAllBytes(@"california-1080-uyva.yuv");
myFrame.Data = Marshal.AllocHGlobal(myUYVYFrame.Length);
Marshal.Copy(myUYVYFrame,0,myFrame.Data,myUYVYFrame.Length);
myFrame.DataLength = myUYVYFrame.Length;
```
Create a loop to start sending the frames

```
while (true)
{
    mySender.Send(myFrame);
}
```

**Receiving Video**

Get an instance of the OMTDiscovery object and wait a few seconds for the sources on the network to be discovered.

```
OMTDiscovery discovery = OMTDiscovery.GetInstance();
Thread.Sleep(5000);
string[] addresses = discovery.GetAddresses();
```

Search through the addresses finding a similar match to the source we created in our Sending Video example.

```
string foundAddress = "";
foreach (string address in addresses)
{
    if (address.Contains("My Sender"))
    {
        foundAddress = address;
        break;
    }
}
```

Create a new OMTReceive object, specifying we wish to receive video and audio, and that the video should be in UYVY format.

```
OMTReceive myReceiver = new OMTReceive(foundAddress, OMTFrameType.Video | OMTFrameType.Audio, OMTPreferredVideoFormat.UYVY, OMTReceiveFlags.None);
```

Create a loop to receive the video within a OMTMediaFrame and write to debug information about the frames that arrive.

```
OMTMediaFrame frame = new OMTMediaFrame();
while (true)
{
    if (myReceiver.Receive(OMTFrameType.Video, 1000, ref frame))
    {
        Debug.WriteLine("Frame Received of size " + frame.Width + "x" + frame.Height + " and timestamp " + frame.Timestamp);
    }
}
```












