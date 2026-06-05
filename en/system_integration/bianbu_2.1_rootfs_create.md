---
sidebar_position: 3
---

# Bianbu 2.1/2.2 ROOTFS Creation

## Environment Requirements

It is recommended to use Ubuntu 20.04/22.04 as the host system, with Docker CE and a customized version of `qemu-user-static` (version 8.0.4, with Vector 1.0 support enabled by default) installed.

### Docker

For Docker CE installation, refer to [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/).

### QEMU

1. **Remove `binfmt-support`**

   The customized `qemu-user-static` conflicts with `binfmt-support` due to:
   - `binfmt-support` uses a legacy SysVinit script: `/etc/init.d/binfmt-support`
   - The customized `qemu-user-static` relies on a systemd unit: `/lib/systemd/system/systemd-binfmt.service`

   Since the SysVinit script runs after the systemd service, it can override the systemd settings, causing conflicts.

   To avoid this, remove `binfmt-support`:

   ```shell
   sudo apt-get purge binfmt-support
   ```

2. **Download the customized qemu**

   ```shell
   wget https://archive.spacemit.com/qemu/qemu-user-static_8.0.4%2Bdfsg-1ubuntu3.23.10.1_amd64.deb
   ```

3. **Install the customized qemu**

   ```shell
   sudo dpkg -i qemu-user-static_8.0.4+dfsg-1ubuntu3.23.10.1_amd64.deb
   ```

4. **Register `qemu-user-static` with the kernel**, so the system (including containers) can directly execute RISC-V binaries:

   ```shell
   sudo systemctl restart systemd-binfmt.service
   ```

5. **Verify** if `qemu-user-static` is registered successfully

   ```shell
   wget https://archive.spacemit.com/qemu/rvv
   chmod a+x rvv
   ./rvv
   ```

   Output like the following indicates success:

   ```
   helloworld
   spacemit
   ```

## Prepare Base ROOTFS

1. **Create a working directory**

   ```shell
   mkdir ~/bianbu-workspace
   ```

2. **Create and start the container**

   ```shell
   docker run --privileged -itd -v ~/bianbu-workspace:/mnt --name build-bianbu-rootfs harbor.spacemit.com/bianbu/bianbu:latest
   ```

3. **Enter the container**

   ```shell
   docker exec -it -w /mnt build-bianbu-rootfs bash
   ```

4. **Install basic tools**

   ```shell
   apt-get update
   apt-get -y install wget uuid-runtime
   ```

5. **Set environment variables**

   - For **Version 2.1**

      ```shell
      export BASE_ROOTFS_URL=https://archive.spacemit.com/bianbu-base/bianbu-base-24.04-base-riscv64.tar.gz
      export BASE_ROOTFS=$(basename "$BASE_ROOTFS_URL")
      export TARGET_ROOTFS=rootfs
      ```

   - For **Version 2.2**

      ```shell
      export BASE_ROOTFS_URL=https://archive.spacemit.com/bianbu-base/bianbu-base-24.04.1-base-riscv64.tar.gz
      export BASE_ROOTFS=$(basename "$BASE_ROOTFS_URL")
      export TARGET_ROOTFS=rootfs
      ```

6. **Download the base rootfs**

   ```shell
   wget $BASE_ROOTFS_URL
   ```

7. **Extract to a specified directory**

   ```shell
   mkdir -p $TARGET_ROOTFS && tar -zxpf $BASE_ROOTFS -C $TARGET_ROOTFS
   ```

8. **Mount system resources into the rootfs**

   ```shell
   mount -t proc /proc $TARGET_ROOTFS/proc
   mount -t sysfs /sys $TARGET_ROOTFS/sys
   mount -o bind /dev $TARGET_ROOTFS/dev
   mount -o bind /dev/pts $TARGET_ROOTFS/dev/pts
   ```

## Essential Configurations

### Configure Repository Sources

1. **Set environment variables**

   ```shell
   export REPO="archive.spacemit.com/bianbu"
   ```

   [Click to view v2.1 release notes](../release_notes/history/bianbu_2.1.md)

   [Click to view v2.2 release notes](../release_notes/bianbu_2.2.md)

2. **Configure `sources.sources`**

   - For **Version 2.1**

      ```shell
      cat <<EOF | tee $TARGET_ROOTFS/etc/apt/sources.list.d/bianbu.sources
      Types: deb
      URIs: https://$REPO/
      Suites: noble/snapshots/v2.1 noble-security/snapshots/v2.1 noble-updates/snapshots/v2.1 noble-porting/snapshots/v2.1 noble-customization/snapshots/v2.1 bianbu-v2.1-updates
      Components: main universe restricted multiverse
      Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
      EOF
      ```

   - For **Version 2.2**

      ```shell
      cat <<EOF | tee $TARGET_ROOTFS/etc/apt/sources.list.d/bianbu.sources
      Types: deb
      URIs: https://$REPO/
      Suites: noble/snapshots/v2.2 noble-security/snapshots/v2.2 noble-updates/snapshots/v2.2 noble-porting/snapshots/v2.2 noble-customization/snapshots/v2.2 bianbu-v2.2-updates
      Components: main universe restricted multiverse
      Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
      EOF
      ```

### Configure DNS

```shell
echo "nameserver 8.8.8.8" >$TARGET_ROOTFS/etc/resolv.conf
```

### Install Hardware-Related Packages

```shell
chroot $TARGET_ROOTFS /bin/bash -c "apt-get update"
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades upgrade"
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install initramfs-tools"
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install bianbu-esos img-gpu-powervr k1x-vpu-firmware k1x-cam spacemit-uart-bt spacemit-modules-usrload opensbi-spacemit u-boot-spacemit linux-generic"
```

### Install Metapackages

Different variants have different metapackages:

- Minimal: `bianbu-minimal`
- Desktop Version: `bianbu-desktop`, `bianbu-desktop-zh`, `bianbu-desktop-en`, `bianbu-desktop-minimal-en`, `bianbu-standard`, `bianbu-development`
- NAS: `bianbu-nas`
- LXQt Desktop Version: `bianbu-desktop-lite`

GNOME, LXQt, and NAS variants are all based on Minimal. It is recommended to install the Minimal meta-package first, then install the corresponding meta-packages.

- **Minimal variant:**

```shell
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install bianbu-minimal"
```

- **GNOME Desktop variant:**

```shell
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install bianbu-minimal"
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install bianbu-desktop bianbu-desktop-zh bianbu-desktop-en bianbu-desktop-minimal-en bianbu-standard bianbu-development"
```

- **LXQt Desktop variant:**

Since the user setup program is not yet fully adapted, you need to manually create a user to enter the desktop environment.

> **Note:** This variant is only supported in Bianbu 2.2.

```shell
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install bianbu-minimal"

chroot $TARGET_ROOTFS /bin/bash -c "apt-get -y install adduser"
chroot $TARGET_ROOTFS /bin/bash -c "adduser --gecos \"\" --disabled-password bianbu"
chroot $TARGET_ROOTFS /bin/bash -c "echo bianbu:bianbu | chpasswd"
chroot $TARGET_ROOTFS /bin/bash -c "usermod -aG adm,cdrom,sudo,plugdev bianbu"

chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install bianbu-desktop-lite"
```

> **Tip:** After installing all packages, you can clean up the cache to reduce the final firmware size:

```shell
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get clean"
```

## Common Configurations

#### Set Locale

```shell
chroot $TARGET_ROOTFS /bin/bash -c "apt-get -y install locales"
chroot $TARGET_ROOTFS /bin/bash -c "echo \"locales locales/locales_to_be_generated multiselect en_US.UTF-8 UTF-8, zh_CN.UTF-8 UTF-8\" | debconf-set-selections"
chroot $TARGET_ROOTFS /bin/bash -c "echo \"locales locales/default_environment_locale select zh_CN.UTF-8\" | debconf-set-selections"
chroot $TARGET_ROOTFS /bin/bash -c "sed -i 's/^# zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/' /etc/locale.gen"
chroot $TARGET_ROOTFS /bin/bash -c "dpkg-reconfigure --frontend=noninteractive locales"
```

#### Set Timezone

```shell
chroot $TARGET_ROOTFS /bin/bash -c "echo 'tzdata tzdata/Areas select Asia' | debconf-set-selections"
chroot $TARGET_ROOTFS /bin/bash -c "echo 'tzdata tzdata/Zones/Asia select Shanghai' | debconf-set-selections"
chroot $TARGET_ROOTFS /bin/bash -c "rm /etc/timezone"
chroot $TARGET_ROOTFS /bin/bash -c "rm /etc/localtime"
chroot $TARGET_ROOTFS /bin/bash -c "dpkg-reconfigure --frontend=noninteractive tzdata"
```

#### Set Time Server

```shell
sed -i 's/^#NTP=.*/NTP=ntp.aliyun.com/' $TARGET_ROOTFS/etc/systemd/timesyncd.conf
```

#### Set Password

```shell
chroot $TARGET_ROOTFS /bin/bash -c "echo root:bianbu | chpasswd"
```

#### Configure Network

- **Minimal**

```shell
cat <<EOF | tee $TARGET_ROOTFS/etc/netplan/01-netcfg.yaml
network:
    version: 2
    renderer: networkd
    ethernets:
        end0:
            dhcp4: true
            dhcp-identifier: mac
        end1:
            dhcp4: true
            dhcp-identifier: mac
EOF
chroot $TARGET_ROOTFS /bin/bash -c "chmod 600 /etc/netplan/01-netcfg.yaml"
```

- **GNOME/LXQt Desktop Version**

```shell
cat <<EOF | tee $TARGET_ROOTFS/etc/netplan/01-network-manager-all.yaml
# Let NetworkManager manage all devices on this system
network:
   version: 2
   renderer: NetworkManager
EOF
chroot $TARGET_ROOTFS /bin/bash -c "chmod 600 /etc/netplan/01-network-manager-all.yaml"
```

> **Note:** Different variants only need to configure their respective files.

## Generate Partition Images

**Note:** After installation and configuration are complete, make sure to unmount first!

```shell
mount | grep "$TARGET_ROOTFS/proc" > /dev/null && umount -l $TARGET_ROOTFS/proc
mount | grep "$TARGET_ROOTFS/sys" > /dev/null && umount -l $TARGET_ROOTFS/sys
mount | grep "$TARGET_ROOTFS/dev/pts" > /dev/null && umount -l $TARGET_ROOTFS/dev/pts
mount | grep "$TARGET_ROOTFS/dev" > /dev/null && umount -l $TARGET_ROOTFS/dev
```

Generate UUIDs and write them to `/etc/fstab`

```shell
UUID_BOOTFS=$(uuidgen)
UUID_ROOTFS=$(uuidgen)
cat >$TARGET_ROOTFS/etc/fstab <<EOF
# <file system>     <dir>    <type>  <options>                          <dump> <pass>
UUID=$UUID_ROOTFS   /        ext4    defaults,noatime,errors=remount-ro 0      1
UUID=$UUID_BOOTFS   /boot    ext4    defaults                           0      2
EOF
```

Move boot to another directory to create separate `bootfs` and `rootfs` partitions:

```shell
mkdir -p bootfs
mv $TARGET_ROOTFS/boot/* bootfs
```

Generate `bootfs.ext4` and `rootfs.ext4`

```shell
mke2fs -d bootfs -L bootfs -t ext4 -U $UUID_BOOTFS bootfs.ext4 "256M"
mke2fs -d $TARGET_ROOTFS -L rootfs -t ext4 -N 524288 -U $UUID_ROOTFS rootfs.ext4 "2048M"
```

At this point, you will have two partition images, `bootfs.ext4` and `rootfs.ext4`, which can be flashed to a board using `fastboot`.

## Create Firmware

See [Firmware Creation Guide](./image.md).
