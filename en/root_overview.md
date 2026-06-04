---
slug: /
sidebar_position: 1
---

# Introduction

**Bianbu** is an operating system deeply optimized for RISC-V architecture processors, built on the Ubuntu community source code, and provides a system foundation for iterative spatiotemporal AI CPUs. Bianbu provides the following version images for developers and users:

- **GNOME Desktop Version:** The native desktop version comes pre-installed with the GNOME Shell desktop environment, Chromium, LibreOffice, MPV, and other applications.
- **LXQt Desktop Version:** A lightweight desktop redesigned and developed based on LXQt, designed for lightweight scenarios with requirements for resource consumption and performance.
- **Minimal Version:** Minimal system version, providing a command-line interface.

## Why Bianbu?

- Provide developers with an operating system deeply optimized for RISC-V architecture processors
- Provide customers with system solutions to accelerate product mass production
- Drive the development of the RISC-V hardware and software ecosystem

## Vision

To empower every individual and every industry around the world with our technology and services.

## System Architecture

![](static/systemarch.png)

## Software Components

Bianbu is composed of the following key components:

- Applications
- Frameworks
- Runtimes
- Libraries
- Linux Kernel
- U-Boot
- OpenSBI

Bianbu manages the packages of these components through the APT source, using the standard Debian package format.

### Applications

- GNOME/LXQt desktop and its common applications
- Remote desktop
- Chromium browser
- LibreOffice suite
- Visual Studio Code
- Docker

### Frameworks

#### Application Frameworks

- Electron
- GTK
- Qt

#### Multimedia Frameworks

- FFmpeg (with Hardware Accelerated)
- GStreamer (with Hardware Accelerated)
- PipeWire

#### Inference Frameworks

- onnxruntime (with Hardware Accelerated)

### Runtimes

- Python
- Java
- Node.js
- Rust

### Libraries

- OpenCV (with RVV Accelerated)
- OpenSSL (with Hardware Accelerated)
- MPP (multimedia processing platform with C API and samples)
- Mesa 3D
- OpenGLES/Vulkan/OpenCL

### Linux Kernel

The Linux kernel is responsible for managing the processor and other hardware resources, providing an interface between users and applications and the hardware. Its main functions include: 
- Interrupt and clock management
- Process scheduling
- Memory management
- File system management
- Device driver management
- Network protocol stack

Supported versions and source links:

- 6.1: [[gitee]](https://gitee.com/spacemit-buildroot/linux-6.1) (EOL)
- 6.6: [[gitee]](https://gitee.com/spacemit-buildroot/linux-6.6) / [[github]](https://github.com/spacemit-com/linux-6.6) (LTS)
- 6.18: [[gitee]](https://gitee.com/spacemit-buildroot/linux-6.18) / [[github]](https://github.com/spacemit-com/linux-6.18)

### U-Boot

U-Boot is a bootloader responsible for initializing specific hardware and loading the Linux kernel image, device tree, and initial RAM filesystem from media (such as SD card, eMMC, SPI Flash, SSD, network).

- Version: u-boot-2022.10
- Source code: [[gitee]](https://gitee.com/spacemit-buildroot/uboot-2022.10) / [[github]](https://github.com/spacemit-com/uboot-2022.10)

### OpenSBI

OpenSBI is an implementation of the Supervisor Binary Interface for RISC-V architecture processors, running in M-mode firmware, providing interfaces for bootloaders, hypervisors, and operating systems to access hardware.

Supported versions and source links:

- 1.3: [[gitee]](https://gitee.com/spacemit-buildroot/opensbi) / [[github]](https://github.com/spacemit-com/opensbi)
- 1.6: [[gitee]](https://gitee.com/spacemit-buildroot/opensbi/tree/k3-br-v1.0.y/) / [[github]](https://github.com/spacemit-com/opensbi/tree/k3-br-v1.0.y)

## Supported Devices

### K1

- BPI-F3
- Milk-V Jupiter
- MUSE Card
- MUSE Pi
- MUSE Pi pro
- MUSE Box
- MUSE Book

### K3

- Pico ITX
- CoM260 Kit

## Release Information

- [Bianbu 1.0](./release_notes/history/bianbu_1.0.md) (End of Life)

- [Bianbu 2.x](./release_notes/bianbu_2.3.md) (K1 Long-Term Maintenance Version)

- [Bianbu 3.x](./release_notes/history/bianbu_3.0.md)

- [Bianbu 4.x](./release_notes/bianbu_4.0.md) (K3 only)

## Feedback & Issues

[https://gitee.com/bianbu/bianbu-docs/issues](https://gitee.com/bianbu/bianbu-docs/issues)
