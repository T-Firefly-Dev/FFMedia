# FFMedia 使用指南

## 什么是 FFMedia？

RK3588 系列芯片拥有超强的视频编解码能力，尤其在多路视频并发处理上表现优异。然而我们在视频处理应用开发时，经常面对 gstreamer、ffmpeg 等通用框架未能充分发挥芯片性能、官方原始 api 太靠近底层、学习成本高、周期长、开发工作量大等问题。

为此，Firefly 基于 Rockchip MPP/RGA 库，开发了一套性能高效、接口简洁、功能完善的视频处理框架——FFMedia。它完整支持市面上主流容器、协议的媒体数据前处理和后处理，同时还支持媒体数据从内存、管道及文件描述符等方式输入和输出，方便对接其他应用和编程语言。

各单元主要组件如下：

- 输入单元：包含 rtsp、rtmp、whep、camera、file 等输入单元；
- 处理单元：包含硬件解码、编码、图像处理及推理单元等支持硬件加速的处理单元；
- 输出单元：包含 rtsp、rtmp、whip、drm display、gb28181、file 等输出单元。

## 功能与特点

### 核心架构

- 模块化架构：整个框架采用 Productor / Consumer 模型，将各个单元都抽象为 ModuleMedia 类；
- 高效内存管理技术：单元之间及硬件的数据交互均使用零拷贝实现。

### 媒体处理能力

- 格式支持：支持 mp4 / mkv / flv / ts 等主流容器格式及 rtsp / rtmp / gb28181 / webrtc 等主流协议的解析与封装；
- 转码和处理：支持视频转码、裁剪、拼接、水印添加等处理；
- 流媒体处理：支持从摄像头、网络流等源拉取媒体流进行实时处理、转发和存储等处理。

### 性能优化

- 低负载和低延迟性：深度优化数据流处理及传递，与 GStreamer 和 FFmpeg 相比，CPU 占用更低，且具备更高的数据实时性；
- 高效 Python 模块：通过 pybind11 实现 C++ 和 Python 之间的无缝互操作性；
- 统一接口：屏蔽和优化复杂的底层操作，为使用者提供高效、统一的接口。

### 平台兼容性

- 芯片级适配：支持 Firefly 平台下所有瑞芯微芯片机器版型；
- 系统支持：支持 Buildroot / Ubuntu / Debian 等不同版本系统。

## 下载源码

拉取源码：
```
git clone https://github.com/Firefly-rk-linux-utils/ffmedia_release.git
```

编译测试：

>参考：[https://github.com/Firefly-rk-linux-utils/ffmedia_release/blob/master/Readme.md](https://github.com/Firefly-rk-linux-utils/ffmedia_release/blob/master/Readme.md)

## 开发接口

所有接口支持 C++ 与 Python 调用。

C++ 语言范式：

```cpp
auto rtsp_c = make_shared<ModuleRtspClient>("rtsp://xxx");
auto ret = rtsp_c->init()
```

Python 语言范式：

```py
rtsp_c = ff_pymedia.ModuleRtspClient("rtsp://xxx")
ret = rtsp_c.init()
```

## 性能测试

测试环境：ITX-3588J

### 低延迟实时流播放

测试播放 H265 的 1080p@30fps 的 RTSP 实时流，使用相关模块：

- RTSP 客户端：采用自实现的轻量级 RTSP 客户端模块；取流一帧耗时 0.03 毫秒左右；
- MPP 解码：基于 MPP 实现的解码模块；解码一帧耗时 1.2 毫秒（多通道模式可低至0.7）左右；
- DRM 显示：基于 DRM 框架实现的显示模块；送显一帧耗时 0.9 毫秒左右。

可计算出直播一路 H265（p帧系列为顺序方式）、1080P 延迟：数据流从网络到解码成 YUV 裸流延迟在1.3 毫秒左右，画面显示还受到屏幕刷新率影响。如 60fps 的屏幕刷新间隔为 16.667 毫秒，可得出显示延迟在 0.9~16.667 毫秒之间。综上，直播一路 1080P 视频的最低延迟为 2.4 毫秒左右。

性能指标如下表所示：

| CPU 占用率 | 内存占用 | 帧处理耗时 |
| :---: | :---: | :---: |
| 1.0% | 72 M | 2.4 ms |

简单测试命令如下：

```bash
./demo rtsp://xxx -d 0
```

测试播放 32 路 H265 的 1080p@30fps rtsp 实时流性能指标如下表所示：

| CPU 占用率 | 内存占用 |
| :---: | :---: |
| 23.5% | 2.8 GB |

简单测试命令如下：

```bash
./demo rtsp://xxx -d 0 -c 32
```

### 实时视频流转码转播

测试将 H265 的 1080p@30fps 的 RTSP 实时流转码成 H264 的 RTSP 流，使用相关模块：

- RTSP 客户端：轻量级 RTSP 客户端模块；取流一帧耗时 0.03 毫秒左右；
- MPP 解码：基于 MPP 实现的解码模块；解码一帧耗时 1.2 毫秒（多通道模式可低至0.7）左右；
- MPP 编码：基于 MPP 实现的编码模块；编码一帧耗时 4.8 毫秒（多通道模式可低至 2.5）左右；
- RTSP 服务端：轻量级 RTSP 服务端模块；推流一帧耗时 0.1 毫秒左右。

可初步估计视频帧从取流、转码最后推流理论耗时为6.3毫秒左右。

性能指标如下表所示：

| CPU 占用率 | 内存占用 | 帧处理耗时 |
| :---: | :---: | :---: |
| 1.0% | 112 M | 6.3 ms |

简单测试命令如下：

```bash
./demo rtsp://xxx -e h264 -p 8554
# 可用demo或其他软件拉取转码后的rtsp流：rtsp://ip:8554/live/0
```
