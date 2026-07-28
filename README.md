# FFMedia User Guide

**English** | [**简体中文**](locales/README.zh-CN.md)

## What is FFMedia?

The RK3588 series chips possess exceptional video encoding and decoding capabilities, especially excelling in concurrent multi-channel video processing. However, during video processing application development, we often face issues where general-purpose frameworks like GStreamer and FFmpeg fail to fully leverage the chip's performance. Furthermore, official raw APIs are often too low-level, resulting in high learning costs, long development cycles, and significant workload.

To address this, Firefly has developed a high-performance, simple, and feature-rich video processing framework based on the Rockchip MPP/RGA libraries—**FFMedia**. It fully supports pre-processing and post-processing of media data for mainstream containers and protocols. Additionally, it supports media data input and output via memory, pipes, and file descriptors, facilitating integration with other applications and programming languages.

The main components of each unit are as follows:

*   **Input Unit:** Includes input units such as RTSP, RTMP, WHEP, Camera, File, etc.
*   **Processing Unit:** Includes hardware-accelerated processing units such as hardware decoding, encoding, image processing, and inference units.
*   **Output Unit:** Includes output units such as RTSP, RTMP, WHIP, DRM Display, GB28181, File, etc.

## Features and Characteristics

### Core Architecture
*   **Modular Architecture:** The entire framework adopts a Producer/Consumer model, abstracting each unit into a `ModuleMedia` class.
*   **Efficient Memory Management:** Zero-copy implementation is used for data interaction between units and hardware.

### Media Processing Capabilities

*   **Format Support:** Supports parsing and encapsulation of mainstream container formats (MP4/MKV/FLV/TS) and protocols (RTSP/RTMP/GB28181/WebRTC).
*   **Transcoding and Processing:** Supports video transcoding, cropping, stitching, watermarking, and other processing tasks.
*   **Streaming Media Processing:** Supports pulling media streams from sources like cameras and network streams for real-time processing, forwarding, and storage.

### Performance Optimization

*   **Low Load and Low Latency:** Deeply optimized data flow processing and transmission. Compared to GStreamer and FFmpeg, it occupies lower CPU usage and offers higher data real-time performance.
*   **Efficient Python Module:** Seamless interoperability between C++ and Python is achieved via `pybind11`.
*   **Unified Interface:** Shields and optimizes complex underlying operations, providing users with efficient and unified interfaces.

### Platform Compatibility

*   **Chip-level Adaptation:** Supports all Rockchip chip machine types on the Firefly platform.
*   **System Support:** Supports different system versions such as Buildroot, Ubuntu, and Debian.

## Download Source Code

Clone the source code:

```bash
git clone https://github.com/Firefly-rk-linux-utils/ffmedia_release.git
```

**Compile and Test:**

> Reference: [https://github.com/Firefly-rk-linux-utils/ffmedia_release/blob/master/Readme.md](https://github.com/Firefly-rk-linux-utils/ffmedia_release/blob/master/Readme.md)

## Development Interfaces

All interfaces support both C++ and Python calls.

**C++ Language Paradigm:**

```cpp
auto rtsp_c = make_shared<ModuleRtspClient>("rtsp://xxx");
auto ret = rtsp_c->init();
```

**Python Language Paradigm:**

```python
rtsp_c = ff_pymedia.ModuleRtspClient("rtsp://xxx")
ret = rtsp_c.init()
```

## Performance Testing

**Test Environment:** ITX-3588J

### Low Latency Real-time Stream Playback

Tested playing a 1080p@30fps H.265 RTSP real-time stream using the following modules:
*   **RTSP Client:** A self-implemented lightweight RTSP client module; fetching a frame takes approximately 0.03ms.
*   **MPP Decode:** A decoding module based on MPP; decoding a frame takes approximately 1.2ms (can be as low as 0.7ms in multi-channel mode).
*   **DRM Display:** A display module based on the DRM framework; displaying a frame takes approximately 0.9ms.

**Calculation:** The latency for live streaming one channel of H.265 (P-frame sequence in order), 1080p is approximately **1.3ms** from network data to decoding into YUV raw stream. Display latency is also affected by the screen refresh rate. For a 60fps screen, the refresh interval is 16.667ms, resulting in a display latency between 0.9ms and 16.667ms. In summary, the minimum latency for live streaming one channel of 1080p video is approximately **2.4ms**.

Performance metrics are shown in the table below:

| CPU Usage | Memory Usage | Frame Processing Time |
| :---: | :---: | :---: |
| 1.0% | 72 MB | 2.4 ms |

Simple test command:

```bash
./demo rtsp://xxx -d 0
```

Performance metrics for testing **32 channels** of 1080p@30fps H.265 RTSP real-time streams:

| CPU Usage | Memory Usage |
| :---: | :---: |
| 23.5% | 2.8 GB |

Simple test command:

```bash
./demo rtsp://xxx -d 0 -c 32
```

### Real-time Video Stream Transcoding and Broadcasting

Tested transcoding a 1080p@30fps H.265 RTSP real-time stream into an H.264 RTSP stream using the following modules:
*   **RTSP Client:** Lightweight RTSP client module; fetching a frame takes ~0.03ms.
*   **MPP Decode:** Decoding module based on MPP; decoding a frame takes ~1.2ms (can be as low as 0.7ms in multi-channel mode).
*   **MPP Encode:** Encoding module based on MPP; encoding a frame takes ~4.8ms (can be as low as 2.5ms in multi-channel mode).
*   **RTSP Server:** Lightweight RTSP server module; pushing a frame takes ~0.1ms.

**Calculation:** The estimated theoretical time consumption for a video frame from fetching, transcoding, to pushing is approximately **6.3ms**.

Performance metrics are shown in the table below:

| CPU Usage | Memory Usage | Frame Processing Time |
| :---: | :---: | :---: |
| 1.0% | 112 MB | 6.3 ms |

Simple test command:

```bash
./demo rtsp://xxx -e h264 -p 8554
# Use demo or other software to pull the transcoded RTSP stream: rtsp://ip:8554/live/0
```
