---
sidebar_position: 4
---

# Open-WebUI

**Open-WebUI** (formerly Ollama WebUI) is an open-source, self-hosted web management tool designed for local or private deployment of large language models (LLMs). Its main goal is to provide a ChatGPT-like interaction experience while supporting offline use and extensive customization.

## Platform Support

| Platform & System     | Acceleration Support |
|-----------------------|----------------------|
| K1 Buildroot          | ❌ Not supported     |
| K1 OpenHarmony 5.0    | ❌ Not supported     |
| K3 Bianbu LXQT/GNOME  | ✅ Supported         |

## Installation

To simplify the process of using LLMs on K1, we provide an `Open-WebUI` deb package for one-click installation.

> **Note:** System version **2.1.1** or later is required.

```shell
sudo apt update
sudo apt install openwebui
```

Wait for the installation to finish.

## Usage

Right-click the `openwebui` desktop icon and choose **Allow Launching**.

![Open-WebUI desktop icon](../static/openwebui.png)

Please refer to [OpenWebUI User Guide](https://forum.spacemit.com/t/topic/185) for details.

## Create a model (optional)

This `.deb` package consists of the Open-WebUI and Ollama container images. To create a model, simply enter the Ollama container and follow the steps below. Here is an example of how to create a model. First of all, access the container shell:

```shell
sudo docker exec -it ollama bash
```

```shell
apt update
apt install wget vim
cd /root
wget https://modelscope.cn/models/second-state/Qwen2.5-0.5B-Instruct-GGUF/resolve/master/Qwen2.5-0.5B-Instruct-Q4_0.gguf
wget https://archive.spacemit.com/spacemit-ai/modelfile/qwen2.5:0.5b.modelfile
ollama create qwen2.5:0.5b -f qwen2.5:0.5b.modelfile
```

Afterward, restart the Ollama container and re-enter Open-WebUI to use the accelerated model:

```shell
sudo docker restart ollama
```
