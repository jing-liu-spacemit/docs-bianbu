---
sidebar_position: 3
---

# Bianbu V4.0 Release Notes

Built from the Ubuntu 26.04 source base.

Bianbu 4.0 repository:

```
Types: deb
URIs: https://archive.spacemit.com/bianbu4/
Suites: resolute resolute-security resolute-updates resolute-backports resolute-porting resolute-customization
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

- Using this repository enables installation of packages released in subsequent V4.0.x versions, such as V4.0.1.
- To download source packages, change `Types: deb` to `Types: deb deb-src`.

## V4.0.1 Release Notes

**Release Date:** 2026-05-29

The corresponding **BSP version:** [V1.0.2](https://spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/k3_buildroot/release_notes/bl-v1.0.y.md)

### LXQt Desktop

- Added appearance settings.
- Added desktop workspace switching.
- Boot screen now supports lower-resolution displays.

### Core Components

**Applications**

- Package repository upgraded to the official Ubuntu 26.04 source.

**Boot**

- Fixed an issue where `reboot fastboot` could not flash firmware.

**Display**

- wlroots: fixed an intermittent issue where the desktop background was lost after suspend/resume.

## V4.0.0 Release Notes

**Release Date:** 2026-04-30

**Note:** Bianbu 4.0 images support K3 only.

The corresponding **BSP version:** [V1.0.0](https://spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/k3_buildroot/release_notes/bl-v1.0.y.md)

### LXQt Desktop

- The in-house `bianbu-control-center` adds new modules for Language & Region, Desktop Settings, Notification Settings, Software Updates, and About.
- Status bar notifications default to Do Not Disturb mode.
- snapshot, VLC media player, Zed, and GNOME System Monitor are installed by default. cheese is no longer included in the default installation.

### Core Components and Applications in Bianbu V4.0

**Applications**

- Chromium 143
- LibreOffice
- VSCodium
- mpv
- fcitx5
- snapshot
- VLC media player
- Zed
- GNOME System Monitor

**Application Frameworks**

- Qt 5.15.8
- Qt 6.10.2
- GTK 3.24.51
- GTK 4.21.6

**Multimedia Frameworks**

- FFmpeg 8.0 (with hardware accelerated)
- GStreamer 1.28.0 (with hardware accelerated)
- PipeWire 1.6.0

**AI Inference Frameworks**

- spacemit-onnxruntime
- llama.cpp-tools-spacemit

**Runtimes**

- Python 3.14.3
- OpenJDK
- Node.js

**Libraries**

- OpenCV 4.14.0
- OpenSSL 3.5.3
- MPP, SpacemiT's multimedia processing platform, with C APIs and sample programs
- Mesa 3D 24.01

**Toolchain**

- GCC 15
