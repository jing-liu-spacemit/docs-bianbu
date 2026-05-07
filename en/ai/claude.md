---
sidebar_position: 6
---

# Claude Code

**Claude Code** is a command-line tool for agentic programming developed by **Anthropic**. It allows developers to interact with Claude AI through a terminal interface for code generation, debugging, refactoring, and other intelligent programming assistance tasks.

## Platform Support

| Platform & System     | Acceleration Support |
|-----------------------|----------------------|
| K1 Buildroot          | ❌ Not supported     |
| K1 OpenHarmony 5.0    | ❌ Not supported     |
| K3 Bianbu LXQT/GNOME  | ✅ Supported         |

## 1. Installation

### 1.1 Install npm

```shell
sudo apt install npm
```

### 1.2 Install Claude Code

```bash
sudo npm i --registry=http://nexus.bianbu.xyz/repository/npmproxy/ -g @anthropic-ai/claude-code
```

Verify the installation:

```bash
claude --version
```

If a version number is printed, the installation is successful:

![Claude version output](../static/claude-version.png)

## 2. Configuration

### 2.1 Set environment variables

**Note**: When creating the API token, select the **claude** group.

```bash
cat >> ~/.bashrc << 'EOF'
export ANTHROPIC_BASE_URL="provider_url"
export ANTHROPIC_AUTH_TOKEN="generated_api_key"
EOF
source ~/.bashrc
```

### 2.2 Configure API key auto-approval

```bash
(cat ~/.claude.json 2>/dev/null || echo 'null') | jq --arg key "${ANTHROPIC_API_KEY: -20}" '(. // {}) | .customApiKeyResponses.approved |= ([.[]?, $key] | unique)' > ~/.claude.json.tmp && mv ~/.claude.json.tmp ~/.claude.json
```

## 3. Usage

Start an interactive Claude Code session:

```shell
claude
```

The startup screen looks like this:

![Claude startup screen](../static/claude-use.png)

Say hello:

![Claude hello example](../static/claude-hello.png)

Run `/model` to switch models:

![Claude model switch](../static/claude-model.png)

In the interactive session, you can:

- Ask programming questions
- Request code generation and optimization
- Perform debugging and refactoring
- Get technical suggestions and best practices

Type `/exit` to leave the session.

## 4. Example

- Enter: "Help me write a program that uses ONNX Runtime to download and run a ResNet50 model for classification, then display the classification result." Claude starts working:

![Claude demo step 1](../static/claude-demo1.png)

- Claude finishes the code and provides operation steps:

![Claude demo step 2](../static/claude-demo2.png)

- Run `run it`. If an exception occurs, Claude starts fixing it automatically:

![Claude demo step 3](../static/claude-demo3.png)

- After the fix, the program runs normally:

![Claude demo step 4](../static/claude-demo4.png)

- Running the program manually also works:

![Claude demo step 5](../static/claude-demo5.png)

## 5. Connect to edge-side AI (simple trial)

### 5.1 Set environment variables

```bash
cat >> ~/.bashrc << 'EOF'
export ANTHROPIC_BASE_URL="http://localhost:8080"
export ANTHROPIC_AUTH_TOKEN="llama"
EOF
source ~/.bashrc
```

### 5.2 Configure API key auto-approval

```bash
(cat ~/.claude.json 2>/dev/null || echo 'null') | jq --arg key "${ANTHROPIC_API_KEY: -20}" '(. // {}) | .customApiKeyResponses.approved |= ([.[]?, $key] | unique)' > ~/.claude.json.tmp && mv ~/.claude.json.tmp ~/.claude.json
```

### 5.3 Start `llama-server`

```bash
llama-server -m Qwen2.5-0.5B-Instruct-Q4_0.gguf -t 8 --host 127.0.0.1 --port 8080 --ctx-size 153600 --n-gpu-layers 0 --batch-size 512 --metrics --no-mmap
```

### 5.4 Run Claude

Start Claude and say hello:

```bash
claude --model qwen2.5:0.5b
```

![Claude with local llama server step 1](../static/claude-llama1.png)

The hello request takes a long time because it prefills 17000+ tokens.

![Claude with local llama server step 2](../static/claude-llama2.png)

Later algorithm-writing requests become noticeably faster, but accuracy is still limited.

![Claude with local llama server step 3](../static/claude-llama3.png)
