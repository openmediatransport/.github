# Open Media Transport

Open Media Transport (OMT) is an open-source network protocol for high performance, low latency video over a local area network (LAN).
It has been designed to support multiple high-definition A/V feeds on a standard gigabit network without any specialised hardware.

It is also completely free and open source!

Looking for Downloads? See here:
https://github.com/openmediatransport/.github/blob/main/DOWNLOADS.md

If you're just looking to get started quickly with developing for OMT. See the [Getting Started](#getting-started) section below.

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
* [Advanced Concepts](#advanced-concepts)
    * [Alpha Channel and Video Formats](#alpha-channel-and-video-formats)
    * [Video Quality Control](#video-quality-control)
    * [Networking / Firewall Requirements](#networking--firewall-requirements)   
* [Developer Guide](#developer-guide)
    * [Getting Started](#getting-started)
    * [Libraries](#libraries)
    * [Quick Examples C#](#quick-examples-c)
    * [More Examples](#more-examples)

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
| 1080p30 | Low | 43mbps |
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

### Receive

Receive is a single end point that connects to a Send source to receive the video, audio and metadata from that source.

### Discovery

Discovery is how a **Send** publishes its name on the network so any Receive can connect to it.

Discovery can use either DNS-SD are multicast UDP discovery protocol, or a dedicated Discovery Server using TCP where multicast is not possible.

Send names are in the format **DEVICENAME (Source Name)** where DEVICENAME is typically either the name of the computer (for computer sources) or a unique device name (in the case of hardware encoders).

Discovery Server documention can be found at https://github.com/openmediatransport/OMTDiscoveryServer

## Advanced Concepts

### Alpha Channel and Video Formats

A variety of video pixel formats are supported including many with alpha channel support.

All of these formats are encoded using the VMX video codec prior to sending over the network.

**Video Formats**
**Send/Receive**
* UYVY = 8bit 4:2:2
* UYVA = 8bit 4:2:2:4 (UYVY followed by alpha plane)
* BGRA = 8bit 4:4:4
* P216 = 16bit Planar 4:2:2
* PA16 = 16bit Planar 4:2:2:4 (P216 followed by a 16bit alpha plane)
  
**Send Only**

These are additional formats implemented as a convenience for senders, and are upconverted to 4:2:2:4 internally:
* YUY2 = 8bit 4:2:2
* NV12 = 8bit Planar 4:2:0
* YV12 = 8bit Planar 4:2:0

**Sending Alpha**

Use the BGRA, UYVA or PA16 pixel formats when sending video with an alpha channel.

The OMTMediaFrame.Flags also needs to be set to Alpha and optionally PreMultiplied if the pixels are premultiplied by alpha.

**Sending 16bit/10bit**

Use the P216 or PA16 pixel formats. To use 10bit, simply ensure the 10bits is placed within the most significant bits of each 16bit pixel.

When these formats are used, OTM will automatically set the HighBitDepth flag on each frame to let the decoder know high bit depth data is available.

**Receiving Alpha**

Set the preferred video format on the receiver to one that includes any of the supports alpha channel formats: BGRA, UYVA or PA16.

Where more than one format is possible, OMT will automatically select the format with alpha only when necessary and set the appropriate flags on the media frame.

**Receiving 16bit/10bit**

Set the preferred video format on the receiver to one that includes P216 or PA16.

Where more than one format is possible, OMT will automatically select the 16bit formats only if the HighBitDepth flag was sent or the sender sent using one of the 16bit formats.

This is an important optimisation, as it takes significantly more system resources to encode and decode 16bit, so it is not used when it is known the source is only 8bit.

**Interlaced Video**

Interlaced video is supported with two interleaved fields stored in a single frame with the top field first.

Remember to set the OMTMediaFrame.Flags to Interlaced when sending interlaced content, as the encoder handles these frames differently.

### Video Quality Control

A Sender can choose from three quality levels for video: Low, Medium and High. (See the [Features / Video](#video) section for the bandwidth requirements of each).

If set to Default, the Sender will initially select Medium quality.

In this default mode, Receivers are permitted to request a preferred quality via the **Suggested Quality** API.

The maximum quality selected amongst all receivers will then be selected by the Sender.

### Networking / Firewall Requirements

Open Media Transport exclusively sends audio and video via TCP. Each sender listens on a single port and each receiver may open up to two connections to that sender for separate audio and video streams.

The sender port range is between 6600 to 6800 by default.

DNS-SD discovery is via the operating system built in service listening on UDP port 5353 (Bonjour on Mac, Avahi on Linux).

Alternatively when using the Discovery Server, all communication is via TCP, with the server listening on port 6399 by default.

## Developer Guide

### Getting Started

The simplest way to start development is to download the prebuild binaries on the libomtnet releases page.
This includes ready to use libraries for both .NET apps and any apps that can use C-style exports.

https://github.com/openmediatransport/libomtnet/releases

**.NET / C# / VB.NET**

Add a reference to libomtnet to your application and make sure to also include the libvmx library in the same directory as the app.

Basic code examples can be found here: [Quick Examples C#](#quick-examples-c)

**C/C++ and other C-style languages**

Include the libomt.h header, include libomt and libvmx with your app.

The header file also includes comments that describe each function in more detail.

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
### More Examples

Other code samples can be found here:
https://github.com/openmediatransport/Examples/tree/main/C%2B%2B









