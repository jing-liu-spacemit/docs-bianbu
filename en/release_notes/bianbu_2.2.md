---
sidebar_position: 1
---

# Bianbu V2.2 Release Notes

Built from **Ubuntu 24.04.1** sources.

Bianbu V2.2 sources:

```
Types: deb
URIs: https://archive.spacemit.com/bianbu/
Suites: noble/snapshots/v2.2 noble-security/snapshots/v2.2 noble-updates/snapshots/v2.2 noble-porting/snapshots/v2.2 noble-customization/snapshots/v2.2 bianbu-v2.2-updates
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

- This source provides packages for all future 2.2.x versions (e.g., 2.2.1), which are hosted in `bianbu-v2.2-updates`.  
- To download source packages, modify `Types: deb` to `Types: deb deb-src`.

## V2.2.1

**Release Date:** August 16, 2025

Corresponding **BSP** version: [v2.2.7](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/bl-v2.2.y.md)

### Major updates to Bianbu OS basic components

- Added support for the Lichee Pi 3A board

## V2.2

**Release Date:** 2025-5-9

The corresponding **BSP version:** [v2.2](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/bl-v2.2.y.md)

### Major updates to LXQt desktop components

**1. New Image - bianbu lxqt desktop version**

- Based on `lxqt 2.1.0`
- Default display protocol: **Wayland**
- New hotkeys in `lxqt-wayland-session`:
  - ctrl + shift + 3: Capture full screen
  - ctrl + shift + 4: Capture selected area
  - F11: Toggle fullscreen for applications
  - Ctrl + Alt + ←/→: Switch between left/right workspaces
  - logo + F1~F4: Jump to specified workspace

### Major updates to GNOME desktop components

There are currently no updates to the GNOME desktop components.

### Major updates to Bianbu OS basic components

**1. Display**

- **wlroots**:
  - Added real-time display of desktop FPS
  - Fixed HDMI red screen issue when replugging dual identical displays

- **raindrop**:
  - Fixed issue where secondary screen or icons might disappear in extended mode

**2. Multimedia**

- **mpp**: Fixed segment fault when multiple encoding occurs
- **pipewire**: Fixed crash caused by locking screen during screen recording
- **gst-plugins-bad1.0**:
  - Added feature for `spacemitsrc` plugin to exit actively during sleep
  - Added frame dropping support for `spacemitsrc`
  - Fixed issue with encoder hanging on long sessions
- **mpv**:
  - Added `dmabuf-wayland` display backend
  - Fixed tearing issue when playing 120fps videos
  - Fixed video freeze issue during aging test
- **k1x-cam**:
  - Added `max96716_max96717_imx556` SerDes output feature
  - Added `sc501ai` output feature and completed ISP tuning
  - Added dual-channel CCIC raw dump feature
  - Fixed `ov5647` abnormal exposure and high noise issue
  - Fixed `dvdd_powen_cnt` mismatch issue during power cycling
- **ffmpeg**:
  - Added feature to decode and output one frame per input frame
  - Adjusted some log levels

**3. Development Tools**

- **gpiozero**: Added support RV4B 40Pin interface

**4. AI**

- **asr-llm**: Supports integration of automatic speech recognition with large language models

**5. BSP**

- **bianbu-esos**:
  - Adapted to `v2.2` kernel
  - Added related license information
