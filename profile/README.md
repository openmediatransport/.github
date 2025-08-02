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

### Audio
* Uncompressed 32 bit floating point audio of up to 32 channels
* Protocol sends only non-silent audio channels to optimize bandwidth consumption

### Network
* Extremely simple TCP protocol
* TCP chosen due to highly efficient, hardware accelerated support across much all network cards and operating systems.

### Metadata
* Timestamped bi-directional side data in XML format

### Discovery
* Sources published on the network using DNS Service Discovery (DNS-SD), with robust OS support on Mac, Linux and Windows.
* Optional Discovery Server where multicast is not available.


