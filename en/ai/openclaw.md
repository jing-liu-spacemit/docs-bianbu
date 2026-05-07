---
sidebar_position: 5
---

# openClaw

## Environment Requirements

- Platform: Linux RISC-V 64
- Node.js: >= 22.12.0
- Package manager: pnpm 10.23.0
- Hardware: SpacemiT K1 or K3 development board

## Install OpenClaw for general users

### 1. Install `nvm` to manage Node.js versions

```bash
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

![nvm installation output](../static/image-1.png)

If `nvm` cannot be found, run the following commands:

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
source ~/.bashrc
```

### 2. Download the npm package (optional)

If you install from <https://www.npmjs.com/package>, you do not need to download the package file manually.

Download link: [openClaw npm package](https://archive.spacemit.com/spacemit-ai/openclaw/openclaw-2026.3.8.1.tgz)

![openClaw package download page](../static/image.png)

### 3. Install Node.js 22

For the K3 development board:

```bash
NVM_NODEJS_ORG_MIRROR=https://archive.spacemit.com/nodejs/k3 nvm install 22
```

For the K1 development board:

```bash
NVM_NODEJS_ORG_MIRROR=https://archive.spacemit.com/nodejs/k1 nvm install 22
```

Then run:

```bash
nvm use 22
```

### 4. Install openclaw

There are two ways to install OpenClaw.

Install OpenClaw from the npm registry:

```bash
npm i -g --legacy-peer-deps @dengxifeng/openclaw --force --registry=https://registry.npmjs.org/
```

Or install it from the downloaded npm package:

```bash
npm install -g --legacy-peer-deps ./openclaw-2026.3.8.1.tgz
```

### 5. Configure openclaw

Run the following command to start the setup process:

```bash
openclaw onboard
```

After installation, configure `openclaw`. The following screenshot shows an example using the Kimi model:

![openClaw onboarding example](../static/image-2.png)

After the configuration is complete, the terminal outputs a local access URL containing a token, for example:

```text
http://127.0.0.1:18789/#token=7229793a7a0a32ff206ab91230ac991221c84301dc3447e6
```

Open this link in a browser to access the OpenClaw Web UI.

The token is shown in the console output after the configuration finishes.

![OpenClaw access URL with token](../static/image-3.png)

## Build, install, and extend openClaw for developers

### Source repository

[https://github.com/SpaceX-mit/openclaw-riscv64](https://github.com/SpaceX-mit/openclaw-riscv64)

### Guides

[https://github.com/SpaceX-mit/openclaw-riscv64/blob/main/BUILD_RUN_GUIDE.md](https://github.com/SpaceX-mit/openclaw-riscv64/blob/main/BUILD_RUN_GUIDE.md)

[https://github.com/SpaceX-mit/openclaw-riscv64/blob/main/PACKAGE_BUILD_COMPLETE.md](https://github.com/SpaceX-mit/openclaw-riscv64/blob/main/PACKAGE_BUILD_COMPLETE.md)

## Official References

[https://github.com/openclaw/openclaw](https://github.com/openclaw/openclaw)
[https://www.openclawcenter.com/](https://www.openclawcenter.com/)
