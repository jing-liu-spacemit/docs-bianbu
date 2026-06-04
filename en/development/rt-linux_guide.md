---
sidebar_position: 13
---

# RT-Linux User Guide

This guide covers how to install the PREEMPT_RT real-time kernel (RT-Linux) on Bianbu for the K3 platform, verify the installation, switch back to the standard kernel, and run real-time performance tests.

## 1. Background

PREEMPT_RT is a set of Linux kernel patches that make previously non-preemptible kernel sections fully preemptible, significantly reducing interrupt response latency. It is designed for applications with strict real-time requirements, such as industrial control, audio/video capture, and robotics.

Bianbu provides pre-built RT kernel packages for the K3 platform. No manual compilation is required — install via `apt`, reboot, and the system switches to the real-time kernel automatically.

**Key differences between the standard kernel and the RT kernel:**

| | Standard Kernel (PREEMPT_VOLUNTARY) | RT Kernel (PREEMPT_RT) |
|---|---|---|
| Preemption model | Voluntary preemption | Fully preemptible |
| Interrupt handling | Runs in hard-IRQ context | Most interrupts are threaded |
| Typical worst-case latency | Hundreds of µs to several ms | Tens of µs |
| Use case | General-purpose computing | Real-time control, low-latency acquisition |

---

## 2. Installing the RT Kernel

### 2.1 Online Installation (Recommended)

Make sure the device has a network connection, then run:

```shell
sudo apt update
sudo apt install linux-image-6.18.3-rt
```

After installation completes, reboot to switch to the RT kernel:

```shell
sudo reboot
```

### 2.2 Verifying the Kernel Version

After rebooting, confirm the RT kernel is active:

```shell
uname -r
```

The output should contain `rt`, for example:

```
6.18.3-rt
```

Also verify the preemption model:

```shell
uname -v | grep -o "PREEMPT_RT"
# or
cat /sys/kernel/realtime
```

An output of `1` confirms the RT kernel is running.

---

## 3. Switching Back to the Standard Kernel

If you need to revert to the standard kernel, follow these steps:

**Step 1 — Reconfigure the standard kernel package to regenerate boot files:**

```shell
sudo dpkg-reconfigure linux-image-6.18.3-generic
```

**Step 2 — Update the boot environment file to point to the standard kernel:**

Edit `/boot/env_k3.txt` and update the kernel entries to:

```
knl_name=vmlinuz-6.18.3-generic
ramdisk_name=initrd.img-6.18.3-generic
dtb_dir=spacemit/6.18.3-generic
```

Or apply all changes in one command:

```shell
sudo sed -i \
  -e 's|^knl_name=.*|knl_name=vmlinuz-6.18.3-generic|' \
  -e 's|^ramdisk_name=.*|ramdisk_name=initrd.img-6.18.3-generic|' \
  -e 's|^dtb_dir=.*|dtb_dir=spacemit/6.18.3-generic|' \
  /boot/env_k3.txt
```

**Step 3 — Reboot to apply the change:**

```shell
sudo reboot
```

After rebooting, run `uname -r`. An output of `6.18.3-generic` confirms you are back on the standard kernel.

---

## 4. Real-Time Performance Testing

Install the `rt-tests` package, which includes `cyclictest` — the standard tool for measuring scheduling latency on real-time systems:

```shell
sudo apt install rt-tests
```

### 4.1 Basic Latency Test

Run a baseline latency test:

```shell
sudo cyclictest -l 100000 -m -S -p 90 -i 200 -h 400 -q
```

Parameter reference:

| Parameter | Description |
|-----------|-------------|
| `-l 100000` | Number of iterations: 100,000 |
| `-m` | Lock memory (`mlockall`) to prevent page-fault latency |
| `-S` | Create a test thread per CPU core (SMP mode) |
| `-p 90` | Real-time priority of the test threads: 90 |
| `-i 200` | Wake-up interval: 200 µs |
| `-h 400` | Collect a latency histogram up to 400 µs |
| `-q` | Quiet mode — print summary only |

### 4.2 Latency Test Under Load

In practice, the system is rarely idle. Testing under CPU and memory pressure gives a more realistic picture of worst-case latency. Open two terminals and run the following commands at the same time:

**Terminal 1 — Apply CPU and memory pressure:**

```shell
# Install the stress tool
sudo apt install stress-ng

# Apply mixed CPU + memory load across all cores
sudo stress-ng --cpu $(nproc) --vm 2 --vm-bytes 256M --timeout 300s
```

**Terminal 2 — Run cyclictest at the same time:**

```shell
sudo cyclictest -l 1000000 -m -S -p 90 -i 200 -h 400 -q
```

### 4.3 Interpreting Results

Typical output looks like this:

```
# Min Latencies: 00004 00004 00004 00004 00005 00005 00005 00005
# Avg Latencies: 00006 00006 00006 00007 00006 00006 00007 00006
# Max Latencies: 00042 00051 00048 00059 00053 00047 00055 00050
```

| Metric | Description | RT kernel reference (K3) |
|--------|-------------|--------------------------|
| Min | Minimum wake-up latency (µs) | < 10 µs |
| Avg | Average wake-up latency (µs) | < 20 µs |
| Max | Maximum wake-up latency (µs) | < 100 µs (idle), < 200 µs (under load) |

> **Note:** Maximum latency is the primary metric for real-time evaluation. If Max consistently exceeds 500 µs under load, check whether any non-preemptible driver modules are affecting real-time behavior.

### 4.4 Other rt-tests Tools

```shell
# Measure thread/process context-switch latency
sudo hackbench -l 1000

# Test timer precision
sudo timerlat -p 90 -u

# List all available tools
ls /usr/bin/cyclictest /usr/bin/hackbench /usr/bin/timerlat /usr/bin/oslat 2>/dev/null
```

---

## 5. Real-Time Application Development Guidelines

When developing real-time applications on the RT kernel, follow these guidelines:

**1. Lock memory** to avoid page-fault latency:

```c
#include <sys/mman.h>
mlockall(MCL_CURRENT | MCL_FUTURE);
```

**2. Set a real-time scheduling policy and priority:**

```c
#include <sched.h>
struct sched_param param = { .sched_priority = 80 };
sched_setscheduler(0, SCHED_FIFO, &param);
```

**3. Avoid blocking system calls** in real-time threads. Functions like `malloc`, `printf`, and file I/O can introduce unbounded latency.

**4. CPU isolation** (optional): Pin real-time threads to dedicated CPU cores to minimize scheduling interference:

```shell
# Add to kernel boot parameters (isolates CPUs 2 and 3)
isolcpus=2,3 nohz_full=2,3 rcu_nocbs=2,3
```

Then use `pthread_setaffinity_np()` in your application to bind threads to the isolated cores.

---

## 6. FAQ

**Q: The system won't boot after installing the RT kernel.**

Follow the steps in **Section 3** to edit `/boot/env_k3.txt` and switch the boot entry back to the standard kernel. Once the system is running normally again, remove the RT kernel package:

```shell
sudo apt remove linux-image-6.18.3-rt
```

**Q: `cyclictest` reports a `permission denied` error.**

Run it with `sudo`, or grant the user real-time scheduling permissions:

```shell
sudo bash -c 'echo "@realtime - rtprio 99" >> /etc/security/limits.conf'
sudo bash -c 'echo "@realtime - memlock unlimited" >> /etc/security/limits.conf'
sudo usermod -aG realtime $USER
```

Log out and back in for the changes to take effect.

**Q: How do I confirm that interrupts are threaded?**

```shell
# List threaded interrupt threads (names starting with irq/)
ps aux | grep "irq/"

# Check the scheduling policy of an interrupt thread
chrt -p $(pgrep -f "irq/1-" | head -1) 2>/dev/null
```

Under the RT kernel, most hardware interrupts run as kernel threads with the `SCHED_FIFO` scheduling policy.
