---
sidebar_position: 12
---

# K3 NOR第二启动介质配置指南

## 1. 背景简介

K3 NOR 启动开发板支持从不同的第二介质加载镜像启动。默认情况下，第二启动介质的优先级顺序为：
**UFS -> SSD -> eMMC -> U盘**

开发板支持将特定的第二介质指定为“最高优先级”，其余介质的相对优先级顺序保持不变。例如，若指定“U盘”为最高优先级，则系统查找启动镜像的顺序将变为：
**U盘 -> UFS -> SSD -> eMMC**

本文将介绍两种修改最高优先级介质的方法。用户可以根据实际场景选择：
* **方法一：修改 EEPROM**（推荐调试使用，即时生效，无需重新编译）
* **方法二：修改 DTS**（推荐定制固件或量产时使用，配置固化在镜像中）

> **说明**：若同时使用了两种方法，系统会以**EEPROM**的配置为准。

---

## 2. 修改方法

### 方法一：通过修改 EEPROM 中特定的 TLV 字段

通过修改 EEPROM 中的 TLV `0x83` 字段可以指定第二启动介质。此操作可在 U-Boot Shell 中进行。

#### U-Boot命令行中修改

**前置条件**：开发板上电启动时，及时按s键中断自动启动过程，进入 U-Boot 命令行环境。

**操作步骤**：
1. 检查当前 EEPROM 状态：执行 `tlv_eeprom`
2. 设置新的第二启动介质（以 SSD 为例）：执行 `tlv_eeprom set 0x83 SSD`
3. 将修改写入存储：执行 `tlv_eeprom write`
4. 重启设备：执行 `reset`

系统重启后，将优先尝试从 SSD 加载镜像。各介质对应的值（区分大小写）如下：
* **SSD**: `SSD`
* **UFS**: `UFS`
* **eMMC**: `MMC`
* **U盘**: `USB`

**操作日志示例**：
```shell
=> tlv_eeprom 
...
[   4.104] Second Boot Device   0x83   3 UFS   <-- 修改前为 UFS
[   4.113] Checksum is valid.

=> tlv_eeprom set 0x83 SSD
=> tlv_eeprom write
Programming passed.

=> tlv_eeprom read
EEPROM data loaded from device to memory.

=> tlv_eeprom 
...
[  22.371] Second Boot Device   0x83   3 SSD   <-- 修改后变为 SSD
[  22.380] Checksum is valid.
=> reset
```

#### 通过fastboot命令修改

**前置条件**：开发板上电启动时，及时按s键中断自动启动过程，进入 U-Boot 命令行环境之后，输入命令`fastboot 0`，开发板进入fastboot模式。

**操作步骤**：
1. 开发板进入fastboot模式，与pc端通过usb连接起来，pc端有fastboot工具
2. pc端输入命令`fastboot oem config:write SecondBootDev@eeprom:SSD`
3. pc端输入命令`fastboot oem config:flush`
4. 重启开发板

系统重启后，将优先尝试从 SSD 加载镜像。各介质对应的值（区分大小写）如下：
* **SSD**: `SSD`
* **UFS**: `UFS`
* **eMMC**: `MMC`
* **U盘**: `USB`

### 方法二：通过修改 DTS (设备树)

如果需要将配置固化在固件中，可以通过修改对应板型的 DTS 文件，重新编译并烧录 U-Boot 固件。

以 `com260` 开发板为例，修改 U-Boot 目录下的 `arch/riscv/dts/k3_com260.dts` 文件。

**操作步骤**：
在 DTS 的根节点 `/ { ... }` 下，添加或修改 `nor-boot-priority-helper` 子节点。假设需要修改 SSD 为最高优先级，配置如下：

```dts
/ {
        nor-boot-priority-helper {
                highest-priority = "ssd";
        };
};
```

`highest-priority` 支持填写的字符串值如下：
* **SSD**: `"ssd"` （或 `"nvme"`）
* **UFS**: `"ufs"` （或 `"scsi"`）
* **eMMC**: `"mmc"`
* **U盘**: `"usb"`

修改完成后，重新编译 U-Boot 并将新固件烧录至开发板即可生效。
