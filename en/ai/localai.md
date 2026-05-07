---
sidebar_position: 6
---

# LocalAI

## Introduction

LocalAI is a complete AI stack for running AI models locally. It is designed to be simple, efficient, and accessible, and provides an OpenAI-compatible API. Users can run large language models (LLMs), image generation, speech transcription, and other AI workloads on consumer-grade hardware, including CPU-only environments, while keeping data private and secure.

This document mainly explains how to compile, install, and use LocalAI from source on the Bianbu platform, and how to add custom inference backends.

## Build and installation steps

### Install system dependencies

```bash
sudo apt update
sudo apt install cmake golang libgrpc-dev make protobuf-compiler-grpc python3-grpc-tools

# Uninstall existing protobuf versions
sudo apt-get remove --purge protobuf-compiler libprotobuf-dev
sudo apt-get autoremove
sudo rm /usr/local/bin/protoc          # remove executable
sudo rm -rf /usr/local/include/google  # remove headers
sudo rm -rf /usr/local/lib/libproto*   # remove libraries
sudo rm -rf /usr/lib/protoc            # other possible path

sudo apt-get install autoconf automake libtool curl make gcc-14 g++-14 unzip

# Switch to /usr/bin, then remove:
# gcc, g++, gcc-ar, gcc-nm, gcc-ranlib,
# riscv64-linux-gnu-gcc, riscv64-linux-gnu-gcc-ar, riscv64-linux-gnu-gcc-nm,
# riscv64-linux-gnu-gcc-ranlib, riscv64-linux-gnu-g++
# and recreate symlinks to the corresponding version 14 binaries
# for example:
# sudo rm /usr/bin/gcc
# sudo ln -s /usr/bin/gcc-14 /usr/bin/gcc

# Download, compile, and install protobuf from source
wget https://github.com/protocolbuffers/protobuf/releases/download/v3.20.3/protobuf-cpp-3.20.3.tar.gz
tar xvzf protobuf-cpp-3.20.3.tar.gz
cd protobuf-3.20.3/
cd cmake
cmake -DCMAKE_INSTALL_PREFIX=/usr/local .
cmake --build . --parallel 8
ctest --verbose
sudo cmake --install .
sudo ldconfig
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
cd ../../

sudo apt install libgrpc++-dev
```

### Compile LocalAI

Download our source package and compile it as follows:

```bash
wget https://archive.spacemit.com/spacemit-ai/localai/localai.tar.gz
tar xvzf localai.tar.gz

# Switch to the project directory
cd localai

# Add the Go bin directory to PATH so tools installed via go install can be run directly
# protoc-gen-go and other tools installed during the build rely on this
export PATH=$PATH:$(go env GOPATH)/bin

# Use the Aliyun proxy to speed up Go module downloads
export GOPROXY=https://mirrors.aliyun.com/goproxy/,direct

# Build
make build

# If the build fails, fix the issue, then run make clean before rebuilding
```

## Add custom inference backends

### Add the RISC-V accelerated llama.cpp backend

We internally modified `llama.cpp` to support RISC-V acceleration and packaged it as a gRPC server binary named `llama-cpp-riscv-spacemit`. Deploy it with the following command:

```bash
cd backend/cpp/spacemit-llama-cpp
bash install.sh
```

Running `install.sh` downloads the prebuilt RISC-V accelerated `llamacpp-grpc-server` binary and quantized models, places them into the proper directories, and creates the related configuration files.

Return to the project root and run `./local-ai --debug` to start LocalAI.
Then open <http://localhost:8080/chat/> in a browser to test it.

If you encounter the following error when invoking the backend:

```text
stderr llama-cpp-riscv-spacemit: error while loading shared libraries: libabsl_synchronization.so.20220623: cannot open shared object file: No such file or directory
```

fix it with:

```bash
sudo apt install libabsl-dev
sudo ln -s /usr/lib/riscv64-linux-gnu/libabsl_synchronization.so /usr/lib/riscv64-linux-gnu/libabsl_synchronization.so.20220623
```

If you want to use our accelerated `llamacpp` backend with other models, you can:

- Download models from <https://archive.spacemit.com/spacemit-ai/gguf/>
- Download the corresponding modelfiles from <https://archive.spacemit.com/spacemit-ai/modelfile/>
- Create a new model configuration file based on `models/spacemit-qwen2.5-0.5b-instruct.yaml`. Update the model name, stop words, and template to match the new model. The template section can refer to the modelfile content.

### Add the RISC-V accelerated ASR backend

The project code is located in `backend/cpp/spacemit-asr-cpp`. Deploy it and restart `local-ai` with:

```bash
cd backend/cpp/spacemit-asr-cpp
bash build.sh

cd ../../../
./local-ai --debug
```

Then test it with:

```bash
# Prepare an audio file named test.wav in advance
curl -X POST http://localhost:8080/v1/audio/transcriptions \
    -H "Content-Type: multipart/form-data" \
    -F "file=@test.wav" \
    -F "model=sensevoicesmall-cpp"
```

### Add the C++ TTS backend

The project code is located in `backend/cpp/matcha-tts-cpp`. Deploy it and restart `local-ai` with:

```bash
cd backend/cpp/matcha-tts-cpp
bash build.sh

cd ../../../
./local-ai --debug
```

Then test it with:

```bash
curl -X POST "http://localhost:8080/tts" \
     -H "Content-Type: application/json" \
     -d '{"input":"你好，今天天气怎么样","model":"matcha-tts-cpp"}' \
     -o output.wav
```
