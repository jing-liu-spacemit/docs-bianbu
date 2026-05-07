---
sidebar_position: 4
---

# SpacemiT AI SDK

## 1. Project Overview

SpacemiT AI SDK is an AI application development kit built for the SpaceMIT K-series chips. In addition to preserving the original multimodal capability components, it further supports higher-level application forms such as Agents, AI Robots, and AI Computers, and provides unified capability encapsulation and access interfaces, including:

- **Computer vision (vision)**: detection, classification, segmentation, tracking, face recognition, pose estimation, and more. Example coverage includes `resnet`, `yolov8`, `yolov11`, `yolov8_seg`, `yolov8_pose`, `bytetrack`, `ocsort`, `yolov5-face`, `arcface`, and `emotion`.
- **Speech**: `VAD` (voice activity detection), `ASR` (automatic speech recognition), and `TTS` (text-to-speech), with runnable demos for standalone verification and joint debugging.
- **Natural language (LLM)**: OpenAI-compatible interface integration such as `llama-server`, with examples like `llm_chat` for quick experience and integration.

![SpacemiT AI SDK architecture](../static/ai-sdk-arch.png)

Each independent AI SDK component provides a common API layer to shield users from complex low-level business details and let them focus on upper-layer application development. At present, the AI SDK mainly provides two external access methods:

- **C++/Python interfaces**: suitable for local integration, secondary development, and embedded deployment. These allow direct invocation of each capability component SDK to build customized applications.
- **HTTP/WS interfaces**: exposed uniformly by the gateway layer, suitable for cross-language, cross-process, or distributed integration scenarios, making it easy to connect lower-level AI capabilities to upper-layer business systems.

> **Note:** This document provides navigation and a quick start for SpacemiT AI SDK from the perspective of top-level applications. Each component is still evolving, and the document will continue to be updated.

## 2. Build and Compilation

### 2.1 Get the source code

This repository uses Git submodules to manage the source code of each component. Recursive cloning is recommended:

```bash
git clone --recurse-submodules https://github.com/spacemit-com/model-zoo.git
# Or use SSH
# git clone --recurse-submodules git@github.com:spacemit-com/model-zoo.git
```

If you have already cloned the main repository but not the submodules:

```bash
cd model-zoo
git submodule update --init --recursive
```

### 2.2 One-command build

This repository aggregates the source code of each component as submodules.

```bash
# Source environment variables
source build/envsetup.sh

# Build applications with one command
# The first build may take some time because dependencies for each component are installed automatically.
m
```

If you want to build only one specific component inside the SDK, enter the corresponding directory and run `mm` instead. You still need to run `source build/envsetup.sh` first:

```bash
source build/envsetup.sh

cd asr && mm
# or
cd vision && mm
cd tts && mm
cd vad && mm
cd llm && mm
```

Build artifacts are usually installed into `output/staging` within the SDK project.

## 3. Run the examples

> **Note:** The examples below assume you have completed the one-command build above. Example executables are installed into `output/staging`.

### 3.1 Computer vision

#### Step 1: Download models

```bash
# Download all vision example models to ~/.cache/models/vision/
bash vision/scripts/download_all_models.sh
```

#### Step 2: Download resource files (images/videos)

```bash
# Download example image/video resources to ~/.cache/assets/
bash vision/scripts/download_assets.sh
```

Default directories:

- Models: `~/.cache/models/vision/`
- Assets: `~/.cache/assets/`

#### Step 3: Run the examples

The following are the run commands for all examples under `vision/examples/`. After running `m`, you can execute them directly from the SDK root directory:

```bash
# Face recognition (similarity)
arcface vision/examples/arcface/config/arcface.yaml

# Face detection
yolov5-face vision/examples/yolov5-face/config/yolov5-face.yaml

# Gesture detection
yolov5_gesture vision/examples/yolov5_gesture/config/yolov5_gesture.yaml

# Object detection
yolov8 vision/examples/yolov8/config/yolov8.yaml
yolov11 vision/examples/yolov11/config/yolov11.yaml

# Pose estimation
yolov8_pose vision/examples/yolov8_pose/config/yolov8_pose.yaml

# Instance segmentation
yolov8_seg vision/examples/yolov8_seg/config/yolov8_seg.yaml

# Image classification / emotion recognition
resnet vision/examples/resnet/config/resnet50.yaml
emotion vision/examples/emotion/config/emotion.yaml

# Multi-object tracking (video/camera)
# These two examples display real-time tracking on screen, so a monitor must be connected.
bytetrack vision/examples/bytetrack/config/bytetrack.yaml
ocsort vision/examples/ocsort/config/ocsort.yaml
```

For optional parameters such as `--image`, `--video`, `--use-camera`, `--output`, and thresholds, refer to the README in each example directory (`vision/examples/*/README.md`) or the [model-zoo-vision README](https://github.com/spacemit-com/model-zoo-vision/blob/main/README.md).

### 3.2 ASR

#### Step 1: Download audio resources (sample audio)

```bash
mkdir -p ~/.cache/models/assets/audio
cd ~/.cache/models/assets/audio
wget https://archive.spacemit.com/spacemit-ai/model_zoo/assets/audio/001_zh_daily_weather.wav
```

More audio resources are available in the [audio resource directory](https://archive.spacemit.com/spacemit-ai/model_zoo/assets/audio).

#### Step 2: Run the examples

```bash
# Recognize a WAV file (example)
asr_file_demo ~/.cache/models/assets/audio/001_zh_daily_weather.wav

# Streaming recognition example; requires an external audio input device
asr_stream_demo -l # list audio device IDs
asr_stream_demo -i 0 -t 5 # -i specifies the input device ID
```

### 3.3 TTS

#### Run the examples

```bash
# Simple synthesis with default parameters
tts_file_demo

# Specify text and backend
tts_file_demo -p "你好世界" -l matcha:zh

# Streaming synthesis example; requires an external audio output device
tts_stream_demo -l # list audio device IDs
tts_stream_demo -o 0 --output 48000 --channels 2 -e matcha:zh-en # -o specifies the output device ID
```

### 3.4 VAD

#### Run the example

```bash
# Built-in simulated audio test
vad_simple_demo
```

### 3.5 LLM

#### Step 1: Download a model (GGUF example)

```bash
mkdir -p ~/.cache/models/llm
cd ~/.cache/models/llm
wget https://archive.spacemit.com/spacemit-ai/model_zoo/llm/qwen2.5-0.5b-instruct-q4_0.gguf
```

To try more models, see the [LLM model directory](https://archive.spacemit.com/spacemit-ai/model_zoo/llm).

#### Step 2: Start an OpenAI-compatible service and run the example

```bash
# Start the service on port 8080
llama-server -m ~/.cache/models/llm/qwen2.5-0.5b-instruct-q4_0.gguf -t 8 --port 8080 &

# Run the SDK example program
llm_chat "你好" "http://localhost:8080/v1" "qwen2.5-0.5b" "You are a helpful assistant." 256
```

If you want to use a cloud or remote OpenAI-compatible service such as DeepSeek, simply replace the second argument with the remote `api_base` and pass the API key through an environment variable, for example:

```bash
export OPENAI_API_KEY=your_cloud_api_key
llm_chat "你好" "https://api.deepseek.com" "deepseek-chat" "You are a helpful assistant." 256
```

## 4. Application Development

### 4.1 C++/Python

Each SpacemiT AI SDK component provides a stable C++ header entry point for application-side integration. Most use a PIMPL design for easier integration and binary distribution, and each component also provides Python bindings and examples. It is recommended to build the artifacts into `output/staging` inside the SDK workspace first, and then perform debugging and deployment based on that staging directory.

- **vision**: `vision_service.h` — see the "Application Development" section in [vision/README.md](https://github.com/spacemit-com/model-zoo-vision/blob/main/README.md)
- **ASR**: `asr_service.h` — see the "Application Development" section in [asr/README.md](https://github.com/spacemit-com/model-zoo-asr/blob/main/README.md)
- **TTS**: `tts_service.h` — see the "Application Development" section in [tts/README.md](https://github.com/spacemit-com/model-zoo-tts/blob/main/README.md)
- **VAD**: `vad_service.h` — see the "Application Development" section in [vad/README.md](https://github.com/spacemit-com/model-zoo-vad/blob/main/README.md)
- **LLM**: `llm_service.h` — see the "Application Development" section in [llm/README.md](https://github.com/spacemit-com/model-zoo-llm/blob/main/README.md)

If your goal is to build a conversational application, it is recommended to start directly from `omni_agent`: first run `voice_chat`, then replace or trim the ASR, TTS, and LLM backends as needed, or integrate MCP tools.

### 4.2 HTTP/WS interfaces

In addition to direct C++/Python SDK integration, the AI SDK also provides unified `HTTP/WS` interfaces through the gateway layer to expose underlying AI capabilities externally. These interfaces are still under active development, and related capabilities and documentation will continue to be improved.

- **HTTP interface**: suitable for request-response scenarios, allowing business systems to access recognition, inference, and generation capabilities through standard REST/HTTP methods.
- **WebSocket interface**: suitable for streaming interaction, real-time push, and long-lived connection scenarios such as speech stream processing, incremental result return, and conversational applications.

## 5. Performance Data

For the overall performance table of each model, refer to [Model Zoo performance data](https://www.spacemit.com/community/document/info?lang=zh&nodepath=ai/compute_stack/ai_compute_stack/modelzoo.md). For detailed test data, test methods, and reproduction instructions for each subdomain, refer to the following component documents:

- [model-zoo-vision/README.md](https://github.com/spacemit-com/model-zoo-vision/blob/main/README.md) — appendix: model performance
- [model-zoo-asr/README.md](https://github.com/spacemit-com/model-zoo-asr/blob/main/README.md) — appendix: performance metrics
- [model-zoo-tts/README.md](https://github.com/spacemit-com/model-zoo-tts/blob/main/README.md) — appendix: performance metrics
- [model-zoo-llm/README.md](https://github.com/spacemit-com/model-zoo-llm/blob/main/README.md) — appendix: model performance
