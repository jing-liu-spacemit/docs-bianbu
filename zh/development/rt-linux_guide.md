---
sidebar_position: 13
---

# RT-Linux 使用指南

本指南介绍如何在 Bianbu（K3 平台）上安装、切换并测试 PREEMPT_RT 实时内核（RT-Linux）。

## 1. 背景简介

PREEMPT_RT 是 Linux 内核的一组实时补丁，通过将内核中的不可抢占区域转换为可抢占区域，显著降低系统中断响应延迟，适用于工业控制、音视频采集、机器人等对实时性要求较高的场景。

Bianbu 为 K3 平台提供了预编译的 RT 内核软件包，用户无需自行编译，直接通过 `apt` 安装后重启即可切换到实时内核。

**普通内核与 RT 内核的主要区别：**

| 对比项 | 普通内核（PREEMPT_VOLUNTARY） | RT 内核（PREEMPT_RT） |
|--------|-----------------------------|-----------------------|
| 抢占模式 | 自愿抢占 | 完全可抢占 |
| 中断处理 | 硬中断上下文执行 | 大部分中断线程化 |
| 典型最大延迟 | 数百 µs～数 ms | 数十 µs 量级 |
| 适用场景 | 通用计算 | 实时控制、低延迟采集 |

---

## 2. 安装 RT 内核

### 2.1 在线安装（推荐）

确保设备已连接网络，执行：

```shell
sudo apt update
sudo apt install linux-image-6.18.3-rt
```

安装完成后重启系统，自动切换到 RT 内核：

```shell
sudo reboot
```

### 2.2 验证当前内核版本

重启后确认已进入 RT 内核：

```shell
uname -r
```

输出中应包含 `rt` 字样，例如：

```
6.18.3-rt
```

同时验证内核抢占模式：

```shell
uname -v | grep -o "PREEMPT_RT"
# 或
cat /sys/kernel/realtime
```

输出 `1` 表示当前运行的是 RT 内核。

---

## 3. 切换回普通内核

如需从 RT 内核切换回普通内核，执行以下步骤：

**步骤一：重新配置普通内核包，更新启动文件**

```shell
sudo dpkg-reconfigure linux-image-6.18.3-generic
```

**步骤二：修改启动环境变量文件，指定普通内核**

编辑 `/boot/env_k3.txt`，将内核相关条目修改为以下内容：

```
knl_name=vmlinuz-6.18.3-generic
ramdisk_name=initrd.img-6.18.3-generic
dtb_dir=spacemit/6.18.3-generic
```

可以使用以下命令一次性完成修改：

```shell
sudo sed -i \
  -e 's|^knl_name=.*|knl_name=vmlinuz-6.18.3-generic|' \
  -e 's|^ramdisk_name=.*|ramdisk_name=initrd.img-6.18.3-generic|' \
  -e 's|^dtb_dir=.*|dtb_dir=spacemit/6.18.3-generic|' \
  /boot/env_k3.txt
```

**步骤三：重启生效**

```shell
sudo reboot
```

重启后执行 `uname -r`，输出 `6.18.3-generic` 即表示已切换回普通内核。

---

## 4. 实时性测试

安装 `rt-tests` 工具包，其中包含 `cyclictest`（实时延迟测试的标准工具）：

```shell
sudo apt install rt-tests
```

### 4.1 快速验证测试

运行 1 分钟的基础延迟测试：

```shell
sudo cyclictest -l 100000 -m -S -p 90 -i 200 -h 400 -q
```

参数说明：

| 参数 | 含义 |
|------|------|
| `-l 100000` | 循环次数 100000 次 |
| `-m` | 锁定内存（mlockall），防止缺页引入延迟 |
| `-S` | 对所有 CPU 核分别创建测试线程（SMP 模式） |
| `-p 90` | 测试线程实时优先级设为 90 |
| `-i 200` | 唤醒间隔 200 µs |
| `-h 400` | 统计延迟直方图，最大 400 µs |
| `-q` | 静默模式，只输出汇总结果 |

### 4.2 压力测试下的延迟测试

实际场景中系统通常处于负载状态，需在压力下评估最大延迟。开两个终端分别执行：

**终端 1：施加 CPU/内存压力**

```shell
# 安装压力工具
sudo apt install stress-ng

# 对所有核施加 CPU + 内存混合压力
sudo stress-ng --cpu $(nproc) --vm 2 --vm-bytes 256M --timeout 300s
```

**终端 2：同时运行 cyclictest**

```shell
sudo cyclictest -l 1000000 -m -S -p 90 -i 200 -h 400 -q
```

### 4.3 结果解读

典型输出如下：

```
# Min Latencies: 00004 00004 00004 00004 00005 00005 00005 00005
# Avg Latencies: 00006 00006 00006 00007 00006 00006 00007 00006
# Max Latencies: 00042 00051 00048 00059 00053 00047 00055 00050
```

| 指标 | 说明 | RT 内核参考值（K3） |
|------|------|-------------------|
| Min | 最小唤醒延迟（µs） | < 10 µs |
| Avg | 平均唤醒延迟（µs） | < 20 µs |
| Max | 最大唤醒延迟（µs） | < 100 µs（空载），< 200 µs（压力下） |

> **注意**：最大延迟（Max Latency）是实时性评估的核心指标。若压力测试下 Max 持续超过 500 µs，需检查是否有不可抢占的驱动模块影响实时性。

### 4.4 其他 rt-tests 工具

```shell
# 测试线程/进程切换延迟
sudo hackbench -l 1000

# 测试定时器精度
sudo timerlat -p 90 -u

# 查看所有可用工具
ls /usr/bin/cyclictest /usr/bin/hackbench /usr/bin/timerlat /usr/bin/oslat 2>/dev/null
```

---

## 5. 实时应用开发注意事项

使用 RT 内核开发实时应用时，建议遵循以下原则：

**1. 锁定内存**，防止缺页延迟：

```c
#include <sys/mman.h>
mlockall(MCL_CURRENT | MCL_FUTURE);
```

**2. 设置实时调度策略和优先级**：

```c
#include <sched.h>
struct sched_param param = { .sched_priority = 80 };
sched_setscheduler(0, SCHED_FIFO, &param);
```

**3. 避免在实时线程中调用可能阻塞的系统调用**，如 `malloc`、`printf`、文件 I/O 等。

**4. CPU 隔离**（可选）：将实时线程绑定到专用 CPU 核，减少调度干扰：

```shell
# 启动时在 GRUB 内核参数中添加（隔离 CPU 2、3）
isolcpus=2,3 nohz_full=2,3 rcu_nocbs=2,3
```

然后在程序中用 `pthread_setaffinity_np()` 将线程绑定到隔离核。

---

## 6. 常见问题

**Q：安装 RT 内核后系统无法启动怎么办？**

按照第 3 节的步骤，修改 `/boot/env_k3.txt` 将启动项切换回普通内核，重启恢复正常后，再执行卸载：

```shell
sudo apt remove linux-image-6.18.3-rt
```

**Q：cyclictest 提示 `permission denied`？**

需要以 root 或 `sudo` 运行，或为用户添加实时调度权限：

```shell
sudo bash -c 'echo "@realtime - rtprio 99" >> /etc/security/limits.conf'
sudo bash -c 'echo "@realtime - memlock unlimited" >> /etc/security/limits.conf'
sudo usermod -aG realtime $USER
```

重新登录后生效。

**Q：如何确认中断是否已线程化？**

```shell
# 查看线程化中断（名称以 irq/ 开头的线程）
ps aux | grep "irq/"

# 或查看中断线程的调度策略
chrt -p $(pgrep -f "irq/1-" | head -1) 2>/dev/null
```

RT 内核下大多数硬中断会以 `SCHED_FIFO` 策略的内核线程形式运行。
