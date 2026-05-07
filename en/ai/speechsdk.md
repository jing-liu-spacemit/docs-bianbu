---
sidebar_position: 18
---

# SPEECH SDK

**Speech Model SDK** (`sm-sdk`) is a comprehensive AI speech service platform that integrates multiple speech processing technologies into a unified solution. It provides voice activity detection (VAD), automatic speech recognition (ASR), translation, text-to-speech (TTS), keyword spotting (KWS), and speaker diarization.

## Core Features

### Voice Activity Detection (VAD)

- **Segment splitting**: identifies speech and silence segments in continuous audio streams.
- **Streaming-friendly**: supports continuous processing by audio chunk, suitable for real-time scenarios.

### Automatic Speech Recognition (ASR)

- **Real-time streaming recognition**: supports low-latency speech-to-text conversion.
- **Multilingual support**: covers Chinese, English, Japanese, Korean, Cantonese, and more.

### Translation

- **Real-time streaming translation**: supports low-latency machine translation.
- **Multilingual support**: supports Chinese, English, Japanese, Korean, Spanish, and more.

### Text-to-Speech (TTS)

- **High-quality synthesis**: natural and fluent speech output.
- **Multilingual support**: supports Chinese and English speech synthesis.
- **Adjustable parameters**: supports speed, pitch, and other tunable parameters.

### Keyword Spotting (KWS)

- **Real-time wake-up**: supports real-time detection of custom keywords.
- **Low power consumption**: optimized detection algorithms for long-running scenarios.
- **Configurable thresholds**: flexible sensitivity settings.

### Speaker Diarization

- **Multi-speaker scenario support**: adds speaker identity labels to recognition results.
- **Can be chained with ASR**: speaker labels can be returned together with transcription results in a pipeline.

## Model List

The `CPU Core` column indicates which type of K3 CPU core the model runs on by default.

| Category | Current Default | Optional Implementations | Default Threads | CPU Core | SpacemiT EP Acceleration |
| --- | --- | --- | --- | --- | --- |
| VAD | `ten_vad` | `ten` / `silero` | 1 | X100 | Not supported |
| ASR | `sensevoice` | `sensevoice` / `qwen3asr` | 2 / 4 | X100 / A100 | Supported |
| Translation | `hy-mt-1.5` | `hy-mt-1.5` / `marian` | 4 / 1 | A100 / X100 | Supported / Not supported |
| TTS | `matcha` | `matcha` / `vits` | 2 | X100 | Supported / Not supported |
| KWS | `wenetspeech` | `wenetspeech` | 1 | X100 | Not supported |
| Diarization | `3dspeaker` | `3dspeaker` | 1 | X100 | Supported |

## Quick Start

`sm-sdk` currently provides two usage methods:

- **Use from source**: suitable for development, debugging, custom pipeline design, and direct integration.
- **Use as a system service**: suitable for unified service management on Bianbu systems.

### Install dependencies

1. **Install `python`**

   ```bash
   sudo apt update
   sudo apt install python
   ```

2. **Install `llama-server`**

   `HY-MT-1.5` and `Qwen3-ASR` depend on a local `llama-server`. If you only use the ONNX model pipeline, you can skip this step.

   Bianbu:

   ```bash
   sudo apt update
   sudo apt install llama.cpp-tools-spacemit
   ```

   Ubuntu:

   ```bash
   sudo apt update
   sudo apt install -y build-essential cmake git libcurl4-openssl-dev
   git clone https://github.com/ggml-org/llama.cpp.git
   cd llama.cpp
   cmake -B build -S . -DGGML_CURL=ON -DCMAKE_INSTALL_PREFIX=$HOME/.local
   cmake --build build -j$(nproc)
   cmake --install build
   export PATH="$HOME/.local/bin:$PATH"
   ```

   To make it persistent, add the following command to your shell profile:

   ```bash
   echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
   source ~/.bashrc
   ```

### Use from source

Source usage currently targets the following environments:

- **Bianbu / RISC-V environment**
- **x86-64 Ubuntu environment**

1. **Create a virtual environment**

   ```bash
   cd path/to/sm-sdk
   curl -LsSf https://astral.sh/uv/install.sh | sh
   uv venv --python 3.13
   source .venv/bin/activate
   ```

2. **Install dependencies**

   Ubuntu:

   ```bash
   uv pip install .
   ```

   Bianbu:

   ```bash
   export UV_EXTRA_INDEX_URL=https://git.spacemit.com/api/v4/projects/33/packages/pypi/simple
   export UV_INDEX_URL=https://mirrors.aliyun.com/pypi/simple
   uv pip install .
   UV_EXTRA_INDEX_URL= uv pip install sherpa-onnx --index-url https://git.spacemit.com/api/v4/projects/81/packages/pypi/simple --no-cache-dir
   ```

3. **Prepare models**

   In Ubuntu environments, ONNX models are usually downloaded automatically during component initialization according to the configuration.

   > Note: `Qwen3-ASR` is not yet adapted to run on Ubuntu.

   On Bianbu, you need to download the pre-quantized model package generated with [xslim](https://github.com/spacemit-com/xslim). Unaccelerated open-source models can also be downloaded automatically according to the configuration.

   ```bash
   cd path/to/sm-sdk
   wget http://159.27.188.198/spacemit-ai/yumeet/yumeet_models.tar.gz
   tar -xf yumeet_models.tar.gz
   ```

4. **Start the service**

   ```bash
   python main.py --start-server
   ```

### Use as a system service

System service usage currently targets only the Bianbu environment.

1. **Install the service**

   When `sm-sdk` is installed for the first time, the required model files are prepared automatically, totaling about 3 GB:

   ```bash
   sudo apt update
   sudo apt install sm-sdk
   ```

2. **Start the service**

   After `apt install sm-sdk`, the service usually starts automatically. If you need manual management, use `systemctl`:

   ```bash
   sudo systemctl start sm-sdk.service
   sudo systemctl stop sm-sdk.service
   sudo systemctl restart sm-sdk.service
   ```

3. **View logs**

   ```bash
   journalctl -u sm-sdk.service
   ```

### Additional notes

Besides the main service, some model backends start independent local processes and use separate ports. However, the main service is still recommended as the unified external entry point.

| Service | Default Port | Description |
| --- | --- | --- |
| Main service | `8060` | Main entry point providing HTTP API, WebSocket API, and FastAPI |
| `HY-MT-1.5` model service | `8062` | Local `llama-server` launched on demand by the translation engine |
| `Qwen3-ASR` model service | `8063` | Local `llama-server` launched on demand by the `Qwen3-ASR` engine |

## Service Configuration

Both usage modes of `sm-sdk` use configuration files to control model selection, pipeline switches, and service parameters.

The configuration directory is resolved in the following order by default:

```bash
# Highest priority: environment variable
$SM_SDK_CONFIG_DIR

# Default directory when running from source
./config/

# Common directory for system service mode
/etc/sm-sdk/

# Fallback directory
/var/lib/sm-sdk/config/
```

The model directory is resolved in the following order by default:

```bash
$SM_SDK_MODEL_DIR
./models/
/var/lib/sm-sdk/models/
```

The configuration files in the current repository serve the following purposes:

- `asr.yaml`: ASR model type, model name, thread count, inference provider, and recognition parameters.
- `audio_stream.yaml`: base sample rate for audio input, reused by other components.
- `diarization.yaml`: speaker diarization model and parameters such as similarity thresholds.
- `keywords.txt`: keyword list for keyword spotting.
- `kws.yaml`: KWS model, threshold, duplicate detection interval, and related settings.
- `pipeline.yaml`: pipeline-level switches, such as whether to enable translation, TTS, punctuation restoration, KWS, and diarization.
- `pseudo_stream.yaml`: offline ASR pseudo-stream output strategy, including inference interval, buffer duration, and stabilization parameters.
- `server.yaml`: FastAPI listening address, port, worker count, and related parameters.
- `sli.yaml`: spoken language identification (SLI) configuration. This capability is not enabled by default in the main WebSocket/HTTP pipeline.
- `terms.json`: terminology correction rules for correcting proper nouns and common ASR misrecognitions.
- `translation.yaml`: translation model configuration, including `llama-server` path, port, context length, thread count, and more.
- `tts.yaml`: TTS model configuration, including language, thread count, provider, and more.
- `vad.yaml`: VAD model and segmentation threshold configuration.

After changing the configuration, restart the service for the changes to take effect:

```bash
# When using from source
python main.py --start-server

# When using as a system service
sudo systemctl restart sm-sdk.service
```

## Service Invocation

### WebSocket

The current real-time streaming endpoint is:

```text
ws://<host>:<port>/ws/stream
```

Protocol summary:

1. After the connection is established, the server first returns a `ready` status message.
2. The client sends raw audio bytes in **16 kHz, mono, PCM16** chunks.
3. The server continuously returns transcription results (`transcript`) or keyword spotting results (`kws`).
4. The client can send control messages to dynamically enable or disable STT, KWS, and translation.
5. After the client sends `{"type": "finish"}`, the server returns a finish message and closes the current processing round.

Typical message formats are shown below.

**Service ready message**

```json
{
    "type": "status",
    "status": "ready",
    "message": "Ready to receive audio"
}
```

**Transcription result message**

```json
{
    "type": "transcript",
    "segment": {
        "text": "你好，世界",
        "speaker_id": 0,
        "speaker_label": "Speaker",
        "start_time": 0.0,
        "end_time": 1.2,
        "language": "zh",
        "translation": "Hello, world",
        "is_final": true,
        "is_pseudo": false
    },
    "is_final": true,
    "timestamp": 1710000000.0
}
```

**Keyword spotting message**

```json
{
    "type": "kws",
    "segment": {
        "keyword": "小迭小迭",
        "confidence": 1.0,
        "start_time": 2.1,
        "end_time": 2.5
    },
    "timestamp": 1710000000.0
}
```

**Control message**

```json
{
    "type": "control",
    "enable_stt": true,
    "enable_kws": false,
    "enable_translation": true
}
```

**Finish message**

```json
{
    "type": "finish"
}
```

> Note: The server currently works in **single-session mode**. When the session is occupied by another WebSocket or REST request, a new connection may receive an error message and be rejected.

### HTTP

After the main service starts, OpenAPI documentation is available at:

```text
http://localhost:8060/docs
```

The main available HTTP interfaces are as follows:

| Method | Path | Request Type | Description |
| --- | --- | --- | --- |
| `POST` | `/v1/audio/transcriptions` | `multipart/form-data` | Upload an audio file and return the transcription result |
| `POST` | `/v1/audio/speech` | `application/json` | Input text and return synthesized speech audio |
| `POST` | `/v1/audio/pipeline` | `multipart/form-data` | Execute ASR → Diarization → Translation → TTS in one pass, depending on configuration |

#### 1. File transcription

```bash
curl -X POST http://localhost:8060/v1/audio/transcriptions \
    -F "file=@audio.wav" \
    -F "model=asr" \
    -F "response_format=json"
```

- `response_format=json`: returns JSON containing `text` and `segments`.
- `response_format=text`: returns plain text only.

#### 2. Text-to-speech

```bash
curl -X POST http://localhost:8060/v1/audio/speech \
    -H "Content-Type: application/json" \
    -d '{
        "input": "你好，欢迎使用 Speech SDK。",
        "response_format": "wav",
        "speed": 1.0,
        "speaker_id": 0
    }' \
    --output output.wav
```

- `response_format` currently supports `wav` and `mp3`.
- The allowed range of `speed` is `0.25 ~ 4.0`.

#### 3. Full audio pipeline

```bash
curl -X POST http://localhost:8060/v1/audio/pipeline \
    -F "file=@audio.wav" \
    -F "model=asr" \
    -F "response_format=json" \
    -F "speed=1.0" \
    -F "speaker_id=0"
```

- `response_format=json`: returns transcription text, translation text, segment information, and time statistics.
- `response_format=wav`: when translation and TTS both execute successfully, returns the synthesized WAV audio directly.

### Call from source code

#### 1. Use `SpeechPipeline` to invoke the speech pipeline

The following example shows how to manually feed audio chunks into the pipeline using `_on_audio_chunk`:

```python
import numpy as np

from src.base.config import load_session_config
from src.core.pipeline import SpeechPipeline


config = load_session_config()
pipeline = SpeechPipeline(config)
pipeline.initialize()


def on_segment(segment):
    print(segment.model_dump())


pipeline.add_callback(on_segment)

pcm16 = open("audio.pcm", "rb").read()
chunk = np.frombuffer(pcm16, dtype=np.int16).astype(np.float32) / 32768.0

pipeline._on_audio_chunk(chunk)
pipeline.flush()
pipeline.reset()
```

Notes:

- The input to `_on_audio_chunk` should be a `float32` mono waveform.
- If the input is not `16 kHz`, it is recommended to resample it first at the application level.
- `flush()` forces the pipeline to output any remaining buffered results after input ends.

#### 2. Use `SpeechComponentManager` to call models directly

The following example shows how to call initialized components directly without going through the full pipeline:

```python
import soundfile as sf

from src.base.config import load_session_config
from src.core.speech_components import SpeechComponentManager


audio, sample_rate = sf.read("audio.wav", dtype="float32")
if audio.ndim > 1:
    audio = audio.mean(axis=1)

config = load_session_config()
components = SpeechComponentManager()
components.configure(config)
components.initialize_all()

asr_result = components.asr.recognize(audio)
print(asr_result.text)

if components.translation is not None:
    translated = components.translation.translate(
            asr_result.text,
            source_lang=asr_result.language,
    )
    print(translated)
```

This approach is suitable for combining capabilities as needed inside an application, for example:

- call ASR only;
- perform ASR first and then translation separately;
- call TTS directly on specified text without going through the speech input pipeline.

## Project Structure

```text
sm-sdk/
├── main.py                    # startup entry
├── src/                       # core implementation
│   ├── server.py              # FastAPI service entry
│   ├── api/                   # HTTP / WebSocket interfaces
│   ├── base/                  # configuration and basic data structures
│   ├── core/                  # speech pipeline and component management
│   ├── models/                # model engine implementations
│   └── utils/                 # download, audio, message, and utility modules
├── config/                    # runtime configuration files
├── docs/                      # internal project documentation
├── models/                    # local model directory
├── debian/                    # Debian / Bianbu packaging files
└── benchmark/                 # performance tests and result records
```

Main module responsibilities:

- `src/api/`: exposes HTTP and WebSocket interfaces externally.
- `src/core/`: handles pipeline orchestration, pseudo-stream output, and component lifecycle management.
- `src/models/`: encapsulates initialization, inference, and reset logic for different model engines.
- `config/`: serves as the single configuration entry point controlling capability switches and model selection.
- `examples/`: provides sample invocations for users and can be used as a reference for secondary development.