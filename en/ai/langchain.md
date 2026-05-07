---
sidebar_position: 12
---

# LangChain

## Platform Support

| Platform & System     | Acceleration Support |
|-----------------------|----------------------|
| K1 Buildroot          | ❌ Not supported     |
| K1 OpenHarmony 5.0    | ❌ Not supported     |
| K3 Bianbu LXQt/GNOME  | ✅ Supported         |

## Installation

### 1. Install Dependencies

Install the required system dependencies:

```bash
sudo apt update
sudo apt install python3-venv libffi-dev libssl-dev pkg-config
```

Install Rust:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain stable
source ~/.cargo/env
rustc --version
```

Sample output as below for successful installation
![](../static/rust-install.png)

### 2. Install LangChain

```bash
python3 -m venv ./langchain_venv
source langchain_venv/bin/activate
pip install langchain  -i https://pypi.tuna.tsinghua.edu.cn/simple
```

## Usage

Activate the virtual environment and verify the installed LangChain version:

```bash
source langchain_venv/bin/activate
python -c "import langchain; print(langchain.__version__)"
```

![](../static/langchain-demo.png)