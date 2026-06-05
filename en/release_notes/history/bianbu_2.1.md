---
sidebar_position: 3
---

# Bianbu V2.1 Release Notes [End of Life]

Bianbu V2.1 will reach end of maintenance on July 31, 2025. We recommend using version 2.2 or later. If you have any questions, please contact us.

Build from **Ubuntu 24.04.1** sources.

Bianbu V2.1 source:

```
Types: deb
URIs: https://archive.spacemit.com/bianbu/
Suites: noble/snapshots/v2.1 noble-security/snapshots/v2.1 noble-updates/snapshots/v2.1 noble-porting/snapshots/v2.1 noble-customization/snapshots/v2.1 bianbu-v2.1-updates
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

- This source provides packages for all future 2.1.x versions (e.g., 2.1.1), which are hosted in `bianbu-v2.1-updates`.  
- To download source packages, modify `Types: deb` to `Types: deb deb-src`.

## V2.1.1

**Release Date:** 2025-3-13

The corresponding **Buildroot version:** [v2.1](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/history/bl-v2.1.y.md)

### Major updates

**AI**

- **openwebui**: Added support for deepseek
- **spacemit-ollama-toolkit**: Added support for deepseek


## V2.1

**Release Date:** 2025-2-11

The corresponding **Buildroot version:** [v2.1](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/history/bl-v2.1.y.md)

### Major updates

**AI**

- **ollama**: Introduced `spacemit-ollama-toolkit`
- **open-webui**: Support for openwebui

**Display**

- **img-gpu-powervr**: Support for calling PVR GPU's OpenCL implementation via `libOpenCL.so.1`; fixed the issue of some PPT slides failing to play
- **mutter**: Fixed occasional black screen issue on the login interface

**Multimedia**

- **GStreamer1.0**: Added `SPACEMIT_GST_DEBUG_DUMP_DOT_DIR` debugging feature, supporting the generation of Pipeline Graphviz charts
- **gst-plugins-base1.0**: Optimized `glsinkbin` display frame rate
- **gst-plugins-bad1.0**: 
  - Added active frame dropping feature for `spacemitsrc` component
  - Fixed the issue of `spacemitsrc` data could not be hardware encoded
- **FFmpeg**: Support for adding watermarks after hardware decoding
- **mpv**: Fixed crash issue after unlocking the screen
- **audacious**: Fixed the issue of two dots appearing on the taskbar

**Applications**

- **chromium-browser-stable**: Fixed several issues, and optimized startup speed
- **chromium**: Upgraded to version 131, hardware video decoding is temporarily not supported
- **LibreOffice**: Adjusted compilation optimization level to `-O2`, improved interaction smoothness
- **GNOME**: Disabled thumbnails, and optimized transition animations
- **xfwm4**: Fixed window manager lag issue in software rendering mode
- **zed**: Added `spacemit-code-forge`
- **plymouth**: Fixed abnormal display issue on the startup screen after flashing

**Development Tools**

- **LLVM**: Support for LLVM 19
- **gcc-13**: Support for `zicbom_zicbop_zicboz_zicond_zfh_zfhmin` extensions
- **sysprof**: Fixed the issue of call stack not being displayed
