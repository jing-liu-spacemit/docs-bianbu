
sidebar_position: 5

# 远程桌面操作指南

本文档介绍如何通过 X86 Ubuntu 24.04 或 Windows 11 上位机，远程桌面操作 MUSE Pi Pro 开发板。

整体流程分为三个阶段：

1. **串口连接**：通过 USB 转 TTL 设备建立串口通信，登录开发板。
2. **联网并获取 IP**：确保开发板接入局域网，获取 IP 地址。
3. **启动 WayVNC 并连接桌面**：在开发板上启动 WayVNC 服务，使用 VNC 客户端连接远程桌面。

> **注意**：根据开发板当前状态，选择对应路径：
> - 若 **首次使用（未初始化）**，进入“首次使用（未初始化）”章节。
> - 若 **已完成初始化**，进入“已完成初始化”章节。

## X86 Ubuntu 24.04

### 接口连接

上位机通过 USB 转 TTL 设备与 MUSE Pi Pro 开发板的 GND、TX、RX 接口连接。示意图如下：

![](static/remote1.png)

**Step 1：** 执行：

```
ls /dev/ttyUSB* 2>/dev/null || ls /dev/ttyACM*
```

![](static/remote2.png)

本示例中，设备为 `/dev/ttyUSB0`

**Step 2：** 安装 `minicom`

```
sudo apt update
sudo apt install minicom
```

**Step 3：** 使用 `minicom` 工具进行连接

```
sudo minicom -D /dev/ttyUSB0 -b 115200
```

![](static/remote3.png)

### Ubuntu 系统登录操作

**Step 1：** 按下开发板复位键，待系统加载至如下界面（若系统已初始化，跳过此步骤，进入下一步）。

![](static/remote4.png)

**Step 2：** 按下回车键，待系统加载至如下界面。

![](static/remote5.png)

**Step 3：** 在提示符后输入用户名 `root`，按下回车键。

**Step 4：** 在密码提示处（**Password:**）输入密码 `bianbu`（输入时终端不显示字符），按下回车键。

系统将完成登录并进入开发板 root 用户。

### 首次使用（未初始化）

#### 阶段一：连接网络并确定 IP 地址

`<remote_ip>`：开发板局域网 IP 地址。

##### 场景一：开发板已接入网线

执行：

```
hostname -I
```

显示 `<remote_ip>`：

![](static/remote6.jpeg)

<a id="ubuntu-wifi-scene2"></a>

##### 场景二：开发板未连接网线，需通过 Wi‑Fi 接入网络

**Step 1：** 执行：

```
ifconfig
```

以下图为例，请记录框选的网卡接口名称（例如：`wlan0`），下方指令需使用。

> **注意**：实际网卡接口名称可能不是 `wlan0`，请以下文中的实际名称为准。

![](static/remote7.png)

**Step 2：** 执行以下指令：

> **注意**：请将命令中的 `wlan0` 替换为实际网卡接口名称。

```
wpa_cli -i wlan0 add_network
```

**Step 3：** 配置 Wi-Fi 名称和密码。

> **注意**：请将下方命令中的 Wi-Fi 名称和密码替换为实际信息。

```
wpa_cli -i wlan0 set_network 0 ssid "\"WiFi名称\""
wpa_cli -i wlan0 set_network 0 psk "\"WiFi密码\""
```

**Step 4：** 启用网络配置。

```
wpa_cli -i wlan0 enable_network 0
```

**Step 5：** 获取 IP 地址。

```
/sbin/dhcpcd wlan0
```

![](static/remote8.png)

**Step 6：** 执行：

```
hostname -I
```

显示 `<remote_ip>`：

![](static/remote9.jpeg)

#### 阶段二：通过 WayVNC 远程桌面连接进行初始化

**Step 1：** 执行：

```
cat > /etc/sddm.conf <<'EOF'
[Theme]
Current=bianbu-theme
  
[General]
DisplayServer=wayland
GreeterEnvironment=QT_WAYLAND_SHELL_INTEGRATION=xdg-shell,WLR_LIBINPUT_NO_DEVICES=1
  
[Wayland]
CompositorCommand=labwc
SessionDir=/usr/share/calamares/wayland-sessions/
  
[Autologin]
User=initer
Session=bianbu-init
Relogin=false
EOF
```

![](static/remote10.png)

**Step 2：** 备份环境配置脚本

```
cp /usr/libexec/start-bianbu-init-env /usr/libexec/start-bianbu-init-env.bak_final
```

**Step 3：** 设置环境变量

```
sed -i '/export QT_QPA_PLATFORM=wayland/a\export LABWC_FALLBACK_OUTPUT=NOOP-fallback\nexport LABWC_VIRTUAL_OUTPUT_SIZE=1920x1080' /usr/libexec/start-bianbu-init-env
```

**Step 4：** 执行：

```
ps aux | grep labwc | grep -v grep
```

获取进程号。（框选位置即为实际进程号位置，以实际显示为准。）

![](static/remote11.png)

**Step 5：** 执行 `kill` 命令并替换为实际进程号：

```
kill 实际进程号
```

![](static/remote12.png)

**Step 6：** 执行：

```
systemctl restart sddm
```

![](static/remote13.png)

**Step 7：** 等待 5 秒后，依次执行：

```
AUTO_UID=$(id -u initer)
```

```
export XDG_RUNTIME_DIR="/run/user/$AUTO_UID"
```

```
export WAYLAND_DISPLAY=$(basename /run/user/$AUTO_UID/wayland-*)
```

```
export QT_QPA_PLATFORM=wayland
export QT_WAYLAND_SHELL_INTEGRATION=xdg-shell
```

![](static/remote14.png)

**Step 8：** 启动 wayvnc

```
XDG_RUNTIME_DIR="$XDG_RUNTIME_DIR" WAYLAND_DISPLAY="$WAYLAND_DISPLAY" wayvnc 0.0.0.0 5900
```

![](static/remote15.png)

> **注意**：使用任何 VNC 客户端连接开发板时，上位机与开发板必须处于同一局域网（例如连接到同一个 Wi-Fi 或路由器）。

在 Ubuntu 上位机开启新的终端。

推荐使用 **Remmina** 客户端，配置方式如下：

**Step 9：** 安装 Remmina

```
sudo apt update
sudo apt install remmina remmina-plugin-rdp remmina-plugin-vnc remmina-plugin-secret
```

![](static/remote16.png)

**Step 10：** 启动 Remmina

```
remmina
```

![](static/remote17.png)

在连接配置中：

协议选择 VNC；

地址输入 `<remote_ip>`:5900；

按下回车进行连接。

![](static/remote18.png)

进入 bianbu 系统初始化界面。

![](static/remote19.png)

进行用户信息配置时，请妥善保管您的账户。建议将账号和密码均设置为 `bianbu`，以便后续操作。配置完毕后，系统即进入初始化阶段，需等待约 10 秒。

### 已完成初始化

#### 阶段一：连接网络并切换账户

按照 [**Ubuntu 系统登录操作**](#ubuntu-系统登录操作) 进行登录。

##### 连接网络

- 若以太网已连接：继续执行后续步骤。
- 若以太网未连接：系统将识别为离线环境，请转入 [**首次使用（未初始化）- 阶段一场景二**](#ubuntu-wifi-scene2) 完成网络配置后，再继续执行后续步骤。


**Step 1：** 执行用户切换操作，切换为普通用户；示例账号为 `bianbu`，实际需替换为前期自行创建的账号名。

```
su - 实际创建的普通用户名
```

![](static/remote20.png)

#### 阶段二：通过 WayVNC 远程桌面连接进入桌面

**Step 1：** 在个人用户目录中执行以下指令：

```
TARGET_USER=$(awk -F: '$3>=1000 && $3<65534 {print $1}' /etc/passwd | head -n1)

sudo tee /etc/sddm.conf > /dev/null <<EOF
[Theme]
Current=bianbu-star

[General]
DisplayServer=wayland
GreeterEnvironment=QT_WAYLAND_SHELL_INTEGRATION=xdg-shell,WLR_LIBINPUT_NO_DEVICES=1
[Wayland]
CompositorCommand=labwc

[Autologin]
User=$TARGET_USER
Session=bianbu-lite
Relogin=false
EOF
```

![](static/remote21.png)

输入当前用户密码。

![](static/remote22.png)

**Step 2：** 备份环境配置脚本

```
sudo cp /usr/bin/startlxqtwayland /usr/bin/startlxqtwayland.clean
```

![](static/remote23.png)

**Step 3：** 设置环境变量

```
sudo sed -i '1a export LABWC_FALLBACK_OUTPUT="NOOP-fallback"\nexport LABWC_VIRTUAL_OUTPUT_SIZE="1920x1080"' /usr/bin/startlxqtwayland
```

![](static/remote24.png)

**Step 4：** 执行：

```
systemctl restart sddm
```

输入密码。

![](static/remote25.png)

**Step 5：** 等待 5 秒后，执行：

```
WAYLAND_SOCKET=$(find /run/user -path "/run/user/0/*" -prune -o -name "wayland-*" -type s -print 2>/dev/null | head -n1)
XDG_RUNTIME_DIR=$(dirname "$WAYLAND_SOCKET")
WAYLAND_DISPLAY=$(basename "$WAYLAND_SOCKET")
```

![](static/remote26.png)

**Step 6：** 启动 wayvnc

```
XDG_RUNTIME_DIR="$XDG_RUNTIME_DIR" WAYLAND_DISPLAY="$WAYLAND_DISPLAY" wayvnc 0.0.0.0 5900
```

![](static/remote27.png)

> **注意**：使用任何 VNC 客户端连接开发板时，上位机与开发板必须处于同一局域网（例如连接到同一个 Wi-Fi 或路由器）。

在 Ubuntu 上位机开启新的终端。

推荐使用 **Remmina** 客户端：

**Step 7：** 启动 Remmina

```
remmina
```

![](static/remote17.png)

在连接配置中：

协议选择 VNC；

地址输入 `<remote_ip>`:5900；

按下回车进行连接。

![](static/remote28.png)

进入桌面。

![](static/remote29.png)

## Windows 11

下载 **MobaXterm** 串口调试工具，官方参考链接：

<https://mobaxterm.mobatek.net/>

下载 **RealVNC** 客户端或者 **TigerVNC** 客户端。

### 接口连接

上位机通过 USB 转 TTL 设备与 MUSE Pi Pro 开发板的 GND、TX、RX 接口连接。示意图如下：

![](static/remote1.png)

以 MobaXterm 工具为例：

1）正确连接串口，并在 **设备管理器** 中确认识别到对应的 COM
端口，如下图所示：

![](static/remote30.jpeg)

2）打开 **MobaXterm**，依次点击 **"Sessions"** → **"New
Session"**，选择连接类型为 **Serial**。

3）在弹出的配置窗口中设置：

**Serial port**：选择识别到的 COM 端口（如 COM7）；

**Speed**：设为 115200；

点击 **OK** 进入串口终端。

![](static/remote31.jpeg)

（MobaXterm 在执行指令时出现行错位、文字重叠属于正常现象。）

### Windows 系统登录操作

按下开发板复位键触发系统加载流程，待系统加载完毕后，按下回车键，随即呈现如下界面：

![](static/remote32.png)

**Step 1：** 按下开发板复位键，待系统加载完成。

**Step 2：** 按下回车键，进入登录界面。

**Step 3：** 在提示符后输入用户名 `root`，按下回车键。

**Step 4：** 在密码提示处（**Password:**）输入密码 `bianbu`（输入时终端不显示字符），按下回车键。

系统将完成登录并进入开发板 root 用户。

### 首次使用（未初始化）

#### 阶段一：连接网络并确定 IP 地址

##### 场景一：开发板已接入网线

**Step 1：** 执行：

```
hostname -I
```

显示 `<remote_ip>`：

![](static/remote33.png)

<a id="windows-wifi-scene2"></a>

##### 场景二：开发板未连接网线，需通过 Wi‑Fi 接入网络

**Step 1：** 执行：

```
ifconfig
```

以下图为例，请记录框选的网卡接口名称（例如：`wlan0`），下方指令需使用。

> **注意**：实际网卡接口名称可能不是 `wlan0`，请以下文中的实际名称为准。

![](static/remote34.png)

**Step 2：** 执行以下指令：

> **注意**：请将命令中的 `wlan0` 替换为实际网卡接口名称。

```
wpa_cli -i wlan0 add_network
```

![](static/remote35.png)

**Step 3：** 配置 Wi-Fi 名称和密码。

> **注意**：请将下方命令中的 Wi-Fi 名称和密码替换为实际信息。

```
wpa_cli -i wlan0 set_network 0 ssid "\"WiFi账号\""
wpa_cli -i wlan0 set_network 0 psk "\"WiFi密码\""
```

![](static/remote36.png)

**Step 4：** 执行：

```
wpa_cli -i wlan0 enable_network 0
```

![](static/remote37.png)

**Step 5：** 等待 5 秒后执行：

```
/sbin/dhcpcd wlan0
```

![](static/remote38.png)

**Step 6：** 执行：

```
hostname -I
```

显示 `<remote_ip>`：

![](static/remote33.png)

#### 阶段二：通过 WayVNC 远程桌面连接进行初始化

**Step 1：** 执行：

```
cat > /etc/sddm.conf <<'EOF'
[Theme]
Current=bianbu-theme

[General]
DisplayServer=wayland
GreeterEnvironment=QT_WAYLAND_SHELL_INTEGRATION=xdg-shell,WLR_LIBINPUT_NO_DEVICES=1

[Wayland]
CompositorCommand=labwc
SessionDir=/usr/share/calamares/wayland-sessions/

[Autologin]
User=initer
Session=bianbu-init
Relogin=false
EOF
```

![](static/remote39.png)

**Step 2：** 备份环境配置脚本

```
cp /usr/libexec/start-bianbu-init-env /usr/libexec/start-bianbu-init-env.bak_final
```

![](static/remote40.png)

**Step 3：** 设置环境变量

```
sed -i '/export QT_QPA_PLATFORM=wayland/a\export LABWC_FALLBACK_OUTPUT=NOOP-fallback\nexport LABWC_VIRTUAL_OUTPUT_SIZE=1920x1080' /usr/libexec/start-bianbu-init-env
```

![](static/remote41.png)

**Step 4：** 执行：

```
ps aux | grep labwc | grep -v grep
```

获取进程号。（框选位置即为实际进程号位置，以实际显示为准。）

![](static/remote42.png)

**Step 5：** 执行 `kill` 命令并替换为实际进程号：

```
kill 实际进程号
```

![](static/remote43.png)

**Step 6：** 执行：

```
systemctl restart sddm
```

![](static/remote44.png)

**Step 7：** 等待 5 秒后，依次执行：

```
AUTO_UID=$(id -u initer)
```

```
export XDG_RUNTIME_DIR="/run/user/$AUTO_UID"
```

```
export WAYLAND_DISPLAY=$(basename /run/user/$AUTO_UID/wayland-*)
```

```
export QT_QPA_PLATFORM=wayland
```

```
export QT_WAYLAND_SHELL_INTEGRATION=xdg-shell
```

![](static/remote45.png)

**Step 8：** 启动 wayvnc

```
XDG_RUNTIME_DIR="$XDG_RUNTIME_DIR" WAYLAND_DISPLAY="$WAYLAND_DISPLAY" wayvnc 0.0.0.0 5900
```

![](static/remote46.png)

**Step 9：** 使用 VNC 客户端连接开发板：

> **注意**：使用任何 VNC 客户端连接开发板时，上位机与开发板必须处于同一局域网（例如连接到同一个 Wi-Fi 或路由器）。

若下载的是 **RealVNC Viewer**，使用 **RealVNC Viewer** 客户端进行连接：

1）启动 VNC Viewer；

2）在连接地址栏输入：`<remote_ip>`（如 192.168.1.100）；

3）回车即可连接至远程桌面界面。

![](static/remote47.png)

点击框选内容。

![](static/remote48.png)

进入 bianbu 系统初始化界面。

![](static/remote49.png)

若下载的是 **TigerVNC**，使用 **TigerVNC** 客户端进行连接：

1）启动 TigerVNC；

2）在 VNC 服务器地址栏输入 IP 地址（如 10.59.240.178）；

3）点击连接即可连接至远程桌面界面。

![](static/remote50.png)

进入 bianbu 系统初始化界面。

![](static/remote51.bmp)

进行用户信息配置时，请妥善保管您的账户。建议将账号和密码均设置为 `bianbu`，以便后续操作。配置完毕后，系统即进入初始化阶段，需等待约 10 秒。

### 已完成初始化

#### 阶段一：连接网络并切换账户

按照 [**Windows 系统登录操作**](#windows-系统登录操作) 进行登录。

##### 连接网络

- 若以太网已连接：继续执行后续步骤。
- 若以太网未连接：系统将识别为离线环境，请转入 [**首次使用（未初始化）- 阶段一场景二**](#windows-wifi-scene2) 完成网络配置后，再继续执行后续步骤。

**Step 1：** 执行用户切换操作，切换为普通用户；示例账号为 `bianbu`，实际需替换为前期自行创建的账号名。

```
su - 实际创建的普通用户名
```

![](static/remote52.png)

#### 阶段二：通过 WayVNC 远程桌面连接进入桌面

**Step 1：** 在个人用户目录中执行以下指令：

```
TARGET_USER=$(awk -F: '$3>=1000 && $3<65534 {print $1}' /etc/passwd | head -n1)

sudo tee /etc/sddm.conf > /dev/null <<EOF
[Theme]
Current=bianbu-star

[General]
DisplayServer=wayland
GreeterEnvironment=QT_WAYLAND_SHELL_INTEGRATION=xdg-shell,WLR_LIBINPUT_NO_DEVICES=1
[Wayland]
CompositorCommand=labwc

[Autologin]
User=$TARGET_USER
Session=bianbu-lite
Relogin=false
EOF
```

![](static/remote53.png)

输入密码并按下回车键。

![](static/remote54.png)

**Step 2：** 备份环境配置脚本

```
sudo cp /usr/bin/startlxqtwayland /usr/bin/startlxqtwayland.clean
```

![](static/remote55.png)

**Step 3：** 设置环境变量

```
sudo sed -i '1a export LABWC_FALLBACK_OUTPUT="NOOP-fallback"\nexport LABWC_VIRTUAL_OUTPUT_SIZE="1920x1080"' /usr/bin/startlxqtwayland
```

![](static/remote56.png)

**Step 4：** 执行：

```
systemctl restart sddm
```

![](static/remote57.png)

输入密码，按下回车键确认。

![](static/remote58.png)

**Step 5：** 执行：

```
WAYLAND_SOCKET=$(find /run/user -path "/run/user/0/*" -prune -o -name "wayland-*" -type s -print 2>/dev/null | head -n1)
```

![](static/remote59.png)

**Step 6：** 执行：

```
XDG_RUNTIME_DIR=$(dirname "$WAYLAND_SOCKET")
```

![](static/remote60.png)

**Step 7：** 执行：

```
WAYLAND_DISPLAY=$(basename "$WAYLAND_SOCKET")
```

![](static/remote61.png)

**Step 8：** 启动 wayvnc

```
XDG_RUNTIME_DIR="$XDG_RUNTIME_DIR" WAYLAND_DISPLAY="$WAYLAND_DISPLAY" wayvnc 0.0.0.0 5900
```

![](static/remote62.png)

**Step 9：** 使用 VNC 客户端连接开发板：

> **注意**：使用任何 VNC 客户端连接开发板时，上位机与开发板必须处于同一局域网（例如连接到同一个 Wi-Fi 或路由器）。

若下载的是 **RealVNC Viewer**，使用 **RealVNC Viewer** 客户端进行连接：

1）启动 VNC Viewer；

2）在连接地址栏输入：`<remote_ip>`（如 192.168.1.100）；

3）回车即可连接至远程桌面界面。

![](static/remote47.png)

进入桌面。

![](static/remote63.png)

若下载的是 **TigerVNC**，使用 **TigerVNC** 客户端进行连接：

1）启动 TigerVNC；

2）在 VNC 服务器地址栏输入 IP 地址（如 10.59.240.178）；

3）点击连接即可连接至远程桌面界面。

![](static/remote50.png)

进入桌面。

![](static/remote64.png)
