---
sidebar_position: 2
---

# Bianbu V2.3 Release Notes

Built from **Ubuntu 24.04.1** sources.

Bianbu V2.3 sources:

```
Types: deb
URIs: https://archive.spacemit.com/bianbu/
Suites: noble/snapshots/v2.3 noble-security/snapshots/v2.3 noble-updates/snapshots/v2.3 noble-porting/snapshots/v2.3 noble-customization/snapshots/v2.3 bianbu-v2.3-updates
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

- This source provides packages for all future 2.3.x versions (e.g., 2.3.1), which are hosted in `bianbu-v2.3-updates`.  
- To download source packages, modify `Types: deb` to `Types: deb deb-src`.

## V2.3.5

**Release Date:** 2026-06-09

The corresponding **BSP version**: [v2.2.9](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/k1_buildroot/release_notes/bl-v2.2.y.md)

### Major updates to LXQt desktop components

1. **UI**
  
   - Refined the Calamares installer interface for compatibility with low-resolution screens.
 
   - Improved the login interface for compatibility with low-resolution screens.
  
   - Optimized workspace switching animation.

2. **Applications**

   - Introduced the in-house developed Bianbu Control Center, featuring modules for appearance, language, desktop, notifications, and about.



## V2.3.3

**Release Date:** 2026-03-02

The corresponding **BSP version**: [v2.2.9](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/k1_buildroot/release_notes/bl-v2.2.y.md)

### Major updates to LXQt desktop components

1. **UI**

   - Refined the Calamares installer interface

   - Improved the login interface

2. **Applications**

   - Introduced the in-house developed Bianbu Control Center, featuring modules for user management, Bluetooth, network, Wi-Fi, power, volume, display, and time settings.

## V2.3.0

**Release Date:** 2025-12-15

The corresponding **BSP version**: [v2.2](https://www.spacemit.com/community/document/info?lang=en&nodepath=software/SDK/buildroot/release_notes/bl-v2.2.y.md)

### Major updates to LXQt desktop components

1. **UI**

   - Optimized status bar user experience
   - Customized Calamares installer interface
   - Added SpacemiT SDDM theme
   - Simplified Panel operations
   - Simplified file browser operations
   - Added Wayland lock screen support
   - Added SpacemiT Qt system theme

2. **Applications**

   - Added support for Yongzhong Office (installable)

3. **Performance**

   - Optimized system startup time
   - LibreOffice now uses Qt6 GPU acceleration
