---
sidebar_position: 12
---

# K3 NOR Second Boot Device Configuration Guide

## 1. Background

K3 NOR boot development boards support loading the boot image from different secondary storage devices. By default, the secondary boot device priority order is:
**UFS → SSD → eMMC → USB drive**

The board allows you to designate a specific device as the highest-priority boot target; the relative order of the remaining devices stays the same. For example, setting "USB drive" as the highest priority changes the boot search order to:
**USB drive → UFS → SSD → eMMC**

This guide describes two ways to change the highest-priority boot device. Choose the one that fits your use case:

- **Method 1: Modify the EEPROM** (recommended for debugging — takes effect immediately, no recompilation needed)
- **Method 2: Modify the DTS** (recommended for custom firmware or production — configuration is baked into the image)

> **Note:** If both methods are configured, the **EEPROM** setting takes precedence.

## 2. Configuration Methods

### Method 1: Modify the TLV Field in the EEPROM

You can specify the secondary boot device by modifying TLV field `0x83` in the EEPROM. This operation is performed in the U-Boot shell.

**Prerequisite:** When the board powers on, press `s` promptly to interrupt the automatic boot process and enter the U-Boot command-line environment.

**Steps:**

1. Check the current EEPROM state: run `tlv_eeprom`
2. Set the new secondary boot device (SSD in this example): run `tlv_eeprom set 0x83 SSD`
3. Write the change to storage: run `tlv_eeprom write`
4. Restart the board: run `reset`

After rebooting, the system will attempt to load the image from SSD first. The supported values for each device (case-sensitive) are:

- **SSD**: `SSD`
- **UFS**: `UFS`
- **eMMC**: `MMC`
- **USB drive**: `USB`

**Example session:**

```shell
=> tlv_eeprom 
...
[   4.104] Second Boot Device   0x83   3 UFS   <-- currently UFS
[   4.113] Checksum is valid.

=> tlv_eeprom set 0x83 SSD
=> tlv_eeprom write
Programming passed.

=> tlv_eeprom read
EEPROM data loaded from device to memory.

=> tlv_eeprom 
...
[  22.371] Second Boot Device   0x83   3 SSD   <-- updated to SSD
[  22.380] Checksum is valid.
=> reset
```

### Method 2: Modify the DTS (Device Tree Source)

To bake the configuration into the firmware, modify the DTS file for your board, then recompile and flash the U-Boot firmware.

Using the CoM260 development board as an example, edit `arch/riscv/dts/k3_com260.dts` in the U-Boot source tree.

**Steps:**
1. Under the root node `/ { ... }` in the DTS file, add or update the `nor-boot-priority-helper` child node. The following example sets SSD as the highest-priority device:

    ```dts
    / {
          nor-boot-priority-helper {
                  highest-priority = "ssd";
        };
    };
    ```

   Supported string values for `highest-priority`:
   - **SSD**: `"ssd"` (or `"nvme"`)
   - **UFS**: `"ufs"` (or `"scsi"`)
   - **eMMC**: `"mmc"`
   - **USB drive**: `"usb"`

2. Recompile U-Boot and flash the new firmware to the board.
