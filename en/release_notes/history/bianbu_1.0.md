---
sidebar_position: 1
---

# Bianbu V1.0 Release Notes [End of Life]

## Bianbu V1.0 End-of-Life (EOL) Notice

Bianbu V1.0 is built on **Ubuntu 23.10**. As the Ubuntu community has officially discontinued maintenance and security updates for Ubuntu 23.10, Bianbu V1.0 no longer has a sustainable upstream support foundation.

Based on **Ubuntu 24.04 LTS** and with **deep optimizations for the RISC-V architecture**, SpacemiT has released the next-generation operating system **Bianbu V2.x**. To align version planning and ensure system security and long-term maintainability, we hereby formally announce:

**Bianbu V1.0 has entered End-of-Life (EOL) status.**

### Lifecycle Support Information

| Version        | End of Support Date | Extended Life-cycle Support (ELS) |
|----------------|---------------------|-----------------------------------|
| Bianbu 1.x.x   | 2024/12/31          | Not provided                      |

### Migration Recommendation

To continue receiving feature updates, security patches, and enhanced RISC-V platform support, **users are strongly encouraged to upgrade to the Bianbu V2.x series**.  
For detailed upgrade instructions, please refer to: **[User Guide — System Upgrade](/en/user_guide/GNOME/upgrade.md)**.

## V1.0 release note

**Release Date:** 2024-5-30

The corresponding **Buildroot version:** [v1.0](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/history/bl-v1.0.y.md)

### Main components

#### Applications

- GNOME 45.0
- Chromium 122
- Nautilus
- Shotwell
- LibreOffice
- mpv
- Rhythmbox
- docker.io 24.0.5

#### Applications Framework

- Electron 29.3.1
- QT 5.15.10
- GTK 4.12.2

#### Multimedia Framework

- FFmpeg 6.0 (with Hardware Accelerated)
- GStreamer 1.22.5 (with Hardware Accelerated)
- PipeWire 0.3.79

#### Inference Framework

- onnxruntime 1.15.1 (with Hardware Accelerated)

#### Runtime

- Python 3.11.6
- OpenJDK 17 (default)
- Node.js

#### Libraries

- OpenCV 4.6.0
- OpenSSL 3.0.10
- MPP, SpacemiT multimedia processing platform, provides C API and sample
- Mesa 3D 22.3.5

### Known issue

- Unable to wake up after suspend for a long time

## V1.0.3

**Release Date:** 2024-6-19

The corresponding **Buildroot version:** [v1.0.3](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/history/bl-v1.0.y.md)

### Major Updates

- Fixed the issue of unable to wake up after a long time of suspend
- Fixed the lag issue in XFCE desktop
- Fixed the crash of the properties dialog box in the file browser
- Fixed the issue of repeated Chinese character input in Chromium

## V1.0.5

**Release Date:** 2024-6-26

The corresponding **Buildroot version:** [v1.0.5](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/history/bl-v1.0.y.md)

### Major Updates

- Fixed the issue of LibreOffice crashing with a low probability when opening Word or Excel files or saving files
- Fixed the issue of cheese becoming unresponsive after the system wakes up from suspend

## V1.0.7

**Release Date:** 2024-7-11

The corresponding **Buildroot version:** [v1.0.7](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/history/bl-v1.0.y.md)

## V1.0.8

**Release Date:** 2024-7-16

The corresponding **Buildroot version:** [v1.0.8](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/history/bl-v1.0.y.md)

## V1.0.9

**Release Date:** 2024-7-20

The corresponding **Buildroot version:** [v1.0.9](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/history/bl-v1.0.y.md)

## V1.0.11

**Release Date:** 2024-8-1

The corresponding **Buildroot version:** [v1.0.11](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/history/bl-v1.0.y.md)

## V1.0.12

**Release Date:** 2024-8-2

The corresponding **Buildroot version:** [v1.0.12](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/history/bl-v1.0.y.md)

### Major Updates

- Fixed object detection and pose tracking startup failed issue

## V1.0.13

**Release Date:** 2024-8-16

The corresponding **Buildroot version:** [v1.0.13](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/history/bl-v1.0.y.md)

### Major Updates

- Added support for boot and shutdown animations
- Added support for installation of the `llm-client` large language model client
- Added support for installation of the `box64`
- Added the universe component
- Fixed issues with incorrect version numbers and repeated online upgrades

## V1.0.14

**Release Date:** 2024-8-31

The corresponding **Buildroot version:** [v1.0.14](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/history/bl-v1.0.y.md)

### Major Updates

- Fixed login issue over serial console when HDMI (configured as primary display) is disconnected
- Fix ssh unsupport compress issue

## V1.0.15

**Release Date:** 2024-9-7

The corresponding **Buildroot version:** [v1.0.15](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/history/bl-v1.0.y.md)

### Major Updates

- Resolved low version issue in `spacemit-uart-bt` package
- Added upgrade support to the in-development **Bianbu 2.0** release
