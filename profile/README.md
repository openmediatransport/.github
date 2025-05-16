# Open Media Transport

Open Media Transport (OMT) is a free, fast, simple protocol for sending and receiving high quality video and audio over a LAN.
OMT is designed to be simple to implement and extremely fast, capable of multiple high definition video streams over a gigabit network with low CPU consumption.

## Features

* Video: 4:2:2 I-frame only video codec with alpha support (4:2:2:4)
* Audio: 32bit floating point audio with up to 32 channels
* Metadata: XML
* Network: TCP protocol

## Design Goals

* Open source and royalty free, including codecs
* Extremely high performance, to enable multiple simultaneous high definition streams on a mid-range CPU
* Low-latency, generally 1 frame of delay
* Optimized support for the two major CPU architectures, x86_64 and ARM64
* Portable code that can run on Windows, Linux and MacOS

## Technical Details

### Video Codec (VMX1)

Video Codec is based on the free codec used in vMix (VMX1) which has been donated under the MIT license for this project.
It has been enhanced with alpha channel support whilst maintaining bitstream compatibility with existing decoders.
VMX1 is already supported in FFMPEG to allow easy editing and processing when OMT streams are recorded to file.

* VMX1 is extremely fast to decode and encode and can handle multiple HD streams over a gigabit LAN on a mid-range CPU (Intel i5, Apple M1, Ryzen 5)
* The VMX1 bitstream format provides extremely high resilience to corrupt data and will gracefully handle errors in the stream.
* Datarate is flexible with broadcast quality achievable at bitrates of around 200Mbps for 1080p60 video.
* A 1/8th preview can be generated from any stream with minimal overhead.

### Audio Codec (FPA1)

Audio is uncompressed 32bit floating audio, with an active channels flag so that any empty (silent) channels are not sent over the network.
Bitrate is therefore fixed at 1536kbps per active channel @ 48000hz

### TCP based protocol

TCP has been chosen due to its simplicity, reliability and high performance due to built-in hardware acceleration on most NICs.
