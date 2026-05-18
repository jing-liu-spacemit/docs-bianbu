---
sidebar_position: 3
---

# Bianbu V4.0 更新说明

基于Ubuntu 26.04源码构建

Bianbu 4.0源：

```
Types: deb
URIs: https://archive.spacemit.com/bianbu4/
Suites: resolute resolute-security resolute-updates resolute-backports resolute-porting resolute-customization
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

- 使用此源即可安装到后续的 V4.0.x（如 V4.0.1）发布的包。
- 如需下载源码，请将`Types: deb`改成`Types: deb deb-src`。

## V4.0 更新说明

发布日期：2026-4-30

**注意：Bianbu 4.0镜像仅支持K3。**

对应的**BSP**版本：[V1.0](https://spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/release_notes/bl-v1.0.y.md)

### LXQt桌面

- 自研应用 bianbu-control-center（配置中心）：新增语言和区域、桌面设置、通知设置、软件更新和关于模块。
- 状态栏通知默认为勿扰模式。
- 默认安装 snapshot（相机）、VLC media player（VLC 媒体播放器）、Zed 和 gnome system monitor（系统监视器），不再默认安装 cheese（茄子）。

### Bianbu v4.0基础组件与应用

**应用**

- Chromium 143
- Libreoffice
- VSCodium
- mpv
- fcitx5
- snapshot（相机）
- VLC media player（VLC 媒体播放器）
- Zed
- gnome system monitor（系统监视器）

**应用框架**

- QT 5.15.8
- QT 6.10.2
- GTK 3.24.51
- GTK 4.21.6

**多媒体框架**

- FFmpeg 8.0 (with Hardware Accelerated)
- GStreamer 1.28.0 (with Hardware Accelerated)
- PipeWire 1.6.0

**AI推理框架**

- spacemit-onnxruntime
- llama.cpp-tools-spacemit

**运行时**

- Python 3.14.3
- OpenJDK
- Node.js

**库**

- OpenCV 4.14.0
- OpenSSL 3.5.3
- MPP，进迭时空多媒体处理平台，提供 C API 和 sample
- Mesa 3D 24.01

**工具链**

- gcc15