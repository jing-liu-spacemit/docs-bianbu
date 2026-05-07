---
sidebar_position: 7
---

# picoClaw

## Platform Support


| Platform & System     | Acceleration Support |
|-----------------------|----------------------|
| K1 Buildroot          | ❌ Not supported     |
| K1 OpenHarmony 5.0    | ❌ Not supported     |
| K3 Bianbu LXQt/GNOME  | ✅ Supported         |

## 1. Install Go

```bash
sudo apt update
sudo apt install golang
```

## 2. Download the source code and build it

```bash
git clone https://github.com/sipeed/picoclaw.git
cd picoclaw
make deps
make build
make install
```

## 3. Configure the environment variable and model

Because `/home/bianbu/.local/bin/picoclaw` is not in the system `PATH`, run:

```bash
sudo cp /home/bianbu/.local/bin/picoclaw /usr/local/bin/
```

## 4. Run `picoclaw onboard`

```bash
pipoclaw onboard
```

## 5. Configure the model

```bash
vim /home/bianbu/.picoclaw/config.json
```

## 6. Verify

```bash
picoclaw agent -m "What is 2+2?"
```

