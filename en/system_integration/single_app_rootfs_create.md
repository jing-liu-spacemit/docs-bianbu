---
sidebar_position: 5
---

# Single Application ROOTFS Creation

This document outlines the steps to customize a ROOTFS based on **Bianbu Minimal Version** for running a single application using the **labwc** display server.


## Environmental requirements and basic ROOTFS creation

To prepare the host environment and generate a basic ROOTFS, refer to [Bianbu 2.1/2.2 ROOTFS Creation](./bianbu_2.1_rootfs_create.md). Once the essential and common configurations are complete, proceed with the Labwc-specific setup below.

## Labwc Configuration

When a desktop login manager is not required, you can directly launch Labwc to provide a Wayland display environment. The process is as follows:

1. Install labwc
2. Configure `getty@tty1.service` to automatically log in to the specified user when the system starts
3. Configure `.bashrc` or `.zshrc` in the user directory to start Labwc upon login  
4. Configure the `autostart` file in the Labwc configuration directory to start the application automatically  

### Install labwc

Run the following commands to update and install labwc in the chroot environment:

```shell
chroot $TARGET_ROOTFS /bin/bash -c "apt-get update"
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades upgrade"
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install labwc"
```

### Configure getty@tty1.service

By modifying the configuration of `getty@tty1.service`, you can enable automatic login for the specified user at system startup. First, update the `ExecStart` setting of the service:

```shell
chroot $TARGET_ROOTFS /bin/bash -c "mkdir -p /usr/lib/systemd/system/getty@tty1.service.d"
chroot $TARGET_ROOTFS /bin/bash -c "cat > /usr/lib/systemd/system/getty@tty1.service.d/override.conf <<'EOF'
[Service]
ExecStart=
ExecStart=-/sbin/agetty --noclear --autologin root %I \$TERM
EOF"
```

### Configure `.bashrc` to automatically start Labwc

Modify the shell configuration file of the root user (the default shell is bash) to start Labwc automatically after login.

Append the Labwc startup command at the end of `/root/.bashrc`, and specify the configuration directory as `/root/.config/labwc`:

```shell
chroot "$TARGET_ROOTFS" /bin/bash -c "echo 'labwc -C /root/.config/labwc' >> /root/.bashrc"
```

### Configure `autostart`

Take `glmark2-es2-wayland` as an example of an application to start automatically.

First, install `glmark2-es2-wayland`:

```shell
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install glmark2-es2-wayland"
```

Then create or modify the `/root/.config/labwc/autostart` file:

```shell
chroot "$TARGET_ROOTFS" /bin/bash -c "mkdir -p /root/.config/labwc"
chroot "$TARGET_ROOTFS" /bin/bash -c "touch /root/.config/labwc/autostart"
chroot "$TARGET_ROOTFS" /bin/bash -c "printf 'glmark2-es2-wayland >/dev/null 2>&1 &\n' > /root/.config/labwc/autostart"
```

To set a wallpaper when Labwc starts, place the wallpaper file under the system directory in rootfs (e.g., `/usr/share/wallpapers/wallpaper.png`), and then modify the `autostart` file. The example below uses a wallpaper from the `bianbu-wallpapers` deb package:

```shell
chroot "$TARGET_ROOTFS" /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install bianbu-wallpapers swaybg"
chroot "$TARGET_ROOTFS" /bin/bash -c "printf 'swaybg -m fill -i /usr/share/backgrounds/SolidIsland-2.png >/dev/null 2>&1 &\n' >> /root/.config/labwc/autostart"
```

> **Tip:** After all packages are installed, run the following command to clean the cache and reduce the final firmware size:

```shell
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get clean"
```

### Configure `rc.xml`

`rc.xml` is the core configuration file for Labwc. It is used to define behaviors, keyboard shortcuts, themes, window rules, and more. For details, refer to the official documentation: [https://github.com/labwc/labwc/blob/master/docs/rc.xml.all](https://github.com/labwc/labwc/blob/master/docs/rc.xml.all).

Below is a basic example of `/root/.config/labwc/rc.xml` that maximizes the `glmark2-es2-wayland` window upon launch:

```shell
chroot "$TARGET_ROOTFS" /bin/bash -c "mkdir -p /root/.config/labwc"
chroot "$TARGET_ROOTFS" /bin/bash -c "cat > /root/.config/labwc/rc.xml <<'EOF'
<?xml version=\"1.0\"?>
<labwc_config>
  <windowRules>
    <windowRule identifier=\"glmark2-es2-wayland\" matchOnce=\"true\">
      <skipTaskbar>yes</skipTaskbar>
      <action name=\"ToggleMaximize\" />
      <action name=\"ToggleAlwaysOnTop\" />
    </windowRule>
  </windowRules>
</labwc_config>
EOF"
```

After completing the above configuration, the system can automatically log in to the root user and start `glmark2-es2-wayland` when it starts.

## Generate Partition Images

See the corresponding section in [bianbu 2.1/2.2 ROOTFS creation](./bianbu_2.1_rootfs_create.md).

## Create Firmware

See [Firmware Creation Guide](./image.md).
