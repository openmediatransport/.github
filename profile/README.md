# Open Media Transport

Open Media Transport (OMT) is an open-source network protocol for high performance, low latency video over a local area network (LAN).
It has been designed to support multiple high-definition A/V feeds on a standard gigabit network without any specialised hardware.

It is also completely free and open source!

## Design Goals

* Completely free, open source, and royalty free under a permissive MIT license
* Simple and easy to implement, with cross platform code available for most platforms
* Optimized to run in software on commodity hardware
* High throughput, allowing for multiple high-definition feeds over a standard gigabit network
* Ultra-fast, low latency codec with less than a frame of delay

## Features

### Video
* 4:2:2 video codec with Alpha offers high quality, extremely low latency and reasonable bitrates over a LAN. 
(1080p60 @ ~200Mbps with HQ profile. Optionally lower bitrate profiles available)
* Existing support for codec in FFmpeg (VMX aka vMix Codec), enabling ease of adoption into existing workflows.
* Royalty and patent free
* Optimized code for both NEON and Intel SIMD instructions, allows 1080p60 encode and decode on a single CPU core.

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
