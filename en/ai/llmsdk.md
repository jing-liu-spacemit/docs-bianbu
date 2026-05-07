<!--
 * Copyright 2022-2023 SPACEMIT. All rights reserved.
 * Use of this source code is governed by a BSD-style license
 * that can be found in the LICENSE file.
 * 
 * @Author: David(qiang.fu@spacemit.com)
 * @Date: 2026-04-03 17:56:08
 * @LastEditTime: 2026-04-03 17:56:46
 * @FilePath: \doc\docs-bianbu\zh\ai\llmsdk.md
 * @Description: 
-->

---
sidebar_position: 17
---

# LLM SDK

## Install the service

```bash
sudo apt update
sudo apt install llm-sdk
```

**Base URL:** `http://localhost:8050`

---

## API Directory

- [System](#system)
- [Models](#models)
- [Chat](#chat)
- [Sessions](#sessions)
- [Knowledge Bases](#knowledge-bases)
- [Config](#config)
- [Assets](#assets)

---

## System

### GET `/`

Root path. Returns the basic API information.

**Request example (curl):**

```bash
curl -X GET http://localhost:8050/
```

**Response example:**

```json
{"message": "llm-sdk LLM Chat API"}
```

---

### GET `/api/health`

Health check.

**Request example (curl):**

```bash
curl -X GET http://localhost:8050/api/health
```

**Response example:**

```json
{"status": "healthy", "app": "llm-sdk"}
```

---

## Models

**Prefix:** `/api/models`

Valid values for `model_type`: `llm` | `embed` | `rerank`

---

### GET `/api/models/list`

Get the full model list for the specified type from the database.

**Query parameters:**

| Parameter  | Type   | Default | Description |
| ---------- | ------ | ------- | ----------- |
| model_type | string | `llm`   | Model type  |

**Request example (curl):**

```bash
curl -X GET http://localhost:8050/api/models/list
```

**Response example:**

```json
{
  "models": [
    {
      "id": 1,
      "name": "Qwen2.5-7B-Q4",
      "path": "/home/user/.cache/llm-sdk/models/llm/model.gguf",
      "download_url": "https://example.com/model.gguf",
      "is_downloaded": true,
      "is_current": true,
      "model_type": "llm"
    }
  ]
}
```

---

### GET `/api/models/current`

Get information about the currently running model.

**Query parameters:**

| Parameter  | Type   | Default | Description |
| ---------- | ------ | ------- | ----------- |
| model_type | string | `llm`   | Model type  |

**Request example (curl):**

```bash
curl -X GET http://localhost:8050/api/models/current
```

**Response example:**

```json
{
  "mode": "llm",
  "model_name": "Qwen2.5-7B-Q4",
  "model_path": "/home/user/.cache/llm-sdk/models/llm/model.gguf",
  "status": "running"
}
```

---

### POST `/api/models/download`

Trigger model download asynchronously in the background and return immediately.

**Request body (JSON):**

```json
{"model_id": 1}
```

**Request example (curl):**

```bash
curl -X POST http://localhost:8050/api/models/download \
  -H "Content-Type: application/json" \
  -d "{\"model_id\": 1}"
```

**Response example:**

```json
{"success": true, "message": "Download started for Qwen2.5-7B-Q4"}
```

If already downloaded:

```json
{"success": true, "already_downloaded": true, "message": "Model already downloaded"}
```

---

### POST `/api/models/download/cancel`

Cancel an ongoing model download.

**Request body (JSON):**

```json
{"model_id": 1}
```

**Request example (curl):**

```bash
curl -X POST http://localhost:8050/api/models/download/cancel \
  -H "Content-Type: application/json" \
  -d "{\"model_id\": 1}"
```

**Response example:**

```json
{"success": true, "message": "Download cancelled for Qwen2.5-7B-Q4"}
```

---

### GET `/api/models/download/status`

Get the download status of the specified model.

**Query parameters:**

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| model_id  | int  | No       | Model ID    |

**Request example (curl):**

```bash
curl -X GET http://localhost:8050/api/models/download/status
```

**Response example:**

```json
{
  "status": "downloading",
  "downloaded": 1048576,
  "total": 4294967296,
  "progress": 0.024,
  "error": null
}
```

Valid `status` values: `not_started` | `downloading` | `completed` | `error`

---

### GET `/api/models/download/status/all` (SSE)

Push the download status of all models through SSE.

**Query parameters:**

| Parameter  | Type   | Required | Description          |
| ---------- | ------ | -------- | -------------------- |
| model_type | string | No       | Filter by model type |

**Request example (curl):**

```bash
curl -X GET http://localhost:8050/api/models/download/status/all
```

**SSE event format:**

```text
data: {"tasks": {"1": {"status": "downloading", "progress": 0.5}}, "timestamp": 12345.0}
```

---

### POST `/api/models/start`

Start the `llama-server` for the specified model.

**Request body (JSON):**

```json
{"model_id": 1}
```

**Request example (curl):**

```bash
curl -X POST http://localhost:8050/api/models/start \
  -H "Content-Type: application/json" \
  -d "{\"model_id\": 1}"
```

**Response example:**

```json
{"success": true, "message": "Server started"}
```

---

### POST `/api/models/start_all`

Start the current model servers for all modes (`llm` / `embed` / `rerank`) in the background.


**Request example (curl):**

```bash
curl -X POST http://localhost:8050/api/models/start_all
```


**Response example:**

```json
{"success": true, "message": "Starting all model servers in background"}
```

---

### POST `/api/models/stop_all`

Stop model servers for all modes (`llm` / `embed` / `rerank`).


**Request example (curl):**

```bash
curl -X POST http://localhost:8050/api/models/stop_all
```


**Response example:**

```json
{"success": true, "message": "All model servers stopped"}
```

---

### POST `/api/models/stop`

Stop the server for the specified model type.


**Request body (JSON):**

```json
{"model_id": 1}
```


**Request example (curl):**

```bash
curl -X POST http://localhost:8050/api/models/stop \
  -H "Content-Type: application/json" \
  -d "{\"model_id\": 1}"
```


**Response example:**

```json
{"success": true, "message": "Server stopped"}
```

---

### POST `/api/models/set_current`

Set the specified model as the current model for its type. This updates the database only and does not start the server.


**Request body (JSON):**

```json
{"model_id": 1}
```


**Request example (curl):**

```bash
curl -X POST http://localhost:8050/api/models/set_current \
  -H "Content-Type: application/json" \
  -d "{\"model_id\": 1}"
```


**Response example:**

```json
{"success": true, "message": "Set Qwen2.5-7B-Q4 as current llm model"}
```

---

### GET `/api/models/server_status`

Query the current server status of the specified mode once.


**Query parameters:**

| Parameter  | Type   | Default | Description |
| ---------- | ------ | ------- | ----------- |
| model_type | string | `llm`   | Model type  |


**Request example (curl):**

```bash
curl -X GET http://localhost:8050/api/models/server_status
```


**Response example:**

```json
{
  "status": "running",
  "model_name": "Qwen2.5-7B-Q4",
  "model_path": "/home/user/.cache/llm-sdk/models/llm/model.gguf",
  "error_message": null
}
```

Valid `status` values: `not_started` | `starting` | `running` | `error` | `stopped`

---

### GET `/api/models/server_status/stream` (SSE)

Push server status through SSE. The refresh interval is adjusted automatically by state: 0.5 s while starting, 3 s while running, and 2 s otherwise.


**Query parameters:**

| Parameter  | Type   | Default | Description |
| ---------- | ------ | ------- | ----------- |
| model_type | string | `llm`   | Model type  |


**Request example (curl):**

```bash
curl -X GET http://localhost:8050/api/models/server_status/stream
```


**SSE event format:**

```text
data: {"status": "running", "model_name": "...", "model_path": "...", "error_message": null, "timestamp": 12345.0}
```

---

### GET `/api/models/all/stream` (SSE)

Unified SSE stream that simultaneously pushes server status and download progress for `llm`, `embed`, and `rerank` modes.


**Request example (curl):**

```bash
curl -X GET http://localhost:8050/api/models/all/stream
```


**SSE event format:**

```json
{
  "modes": {
    "llm": {
      "server_status": {"status": "running", "model_name": "...", "model_path": "...", "error_message": null},
      "download_tasks": {}
    },
    "embed": { "..." : "..." },
    "rerank": { "..." : "..." }
  },
  "timestamp": 12345.0
}
```

---

### GET `/api/models/get_param`

Get model parameters for the specified type, including server-side and client-side parameters.


**Query parameters:**

| Parameter  | Type   | Default | Description |
| ---------- | ------ | ------- | ----------- |
| model_type | string | `llm`   | Model type  |


**Response example (`llm`):**

```json
{
  "context_size": 4096,
  "threads": 4,
  "gpu_layers": 0,
  "batch_size": 512,
  "temperature": 0.7,
  "repeat_penalty": 1.1,
  "max_tokens": 2048
}
```

Additional fields for `embed`: `normalize`, `truncate`; additional fields for `rerank`: `top_n`, `return_documents`.


**Request example (curl):**

```bash
curl -X GET http://localhost:8050/api/models/get_param
```

---

### POST `/api/models/update_param`

Update model parameters and save them to the database. Changes to server parameters restart the server automatically; client parameters take effect immediately.


**Query parameters:**

| Parameter  | Type   | Required | Description |
| ---------- | ------ | -------- | ----------- |
| model_type | string | Yes      | Model type  |


**Request body (JSON, all fields optional):**

```json
{
  "context_size": 4096,
  "threads": 8,
  "gpu_layers": 0,
  "batch_size": 512,
  "temperature": 0.8,
  "repeat_penalty": 1.1,
  "max_tokens": 2048,
  "normalize": true,
  "truncate": true,
  "top_n": 10,
  "return_documents": true
}
```


**Request example (curl):**

```bash
curl -X POST http://localhost:8050/api/models/update_param
```


**Response example:**

```json
{
  "success": true,
  "message": "Server parameters updated (server auto-restarted if running)",
  "server_params_changed": true,
  "client_params_changed": false
}
```

---

### POST `/api/models/reset_param`

Reset model parameters of the specified type to defaults. If the server is running, it restarts automatically.


**Query parameters:**

| Parameter  | Type   | Default | Description |
| ---------- | ------ | ------- | ----------- |
| model_type | string | `llm`   | Model type  |


**Request example (curl):**

```bash
curl -X POST http://localhost:8050/api/models/reset_param
```


**Response:** returns the full reset parameter set, using the same format as `GET /api/models/get_param`.

---

## Chat

**Prefix:** `/api`

---

### POST `/api/chat`

Send a chat message. Both streaming (SSE) and non-streaming responses are supported.

Two modes are supported:

- **Standard conversation**: `ConversationChain` with conversation history.
- **RAG conversation**: automatically switches to `RAGChain` when `kb_ids` is provided.


**Request body (JSON):**

```json
{
  "message": {
    "role": "user",
    "content": "你好，请介绍一下自己"
  },
  "session_id": 1,
  "stream": true,
  "kb_ids": null,
  "temperature": null,
  "repeat_penalty": null,
  "max_tokens": null,
  "model_type": "llm"
}
```

| Field          | Type     | Required | Description                                         |
| -------------- | -------- | -------- | --------------------------------------------------- |
| message        | object   | Yes      | User message containing `role` and `content`        |
| session_id     | int      | Yes      | Session ID                                          |
| stream         | bool     | No       | Whether to use streaming response, default `true`   |
| kb_ids         | string[] | No       | Knowledge base ID list; enables RAG when provided   |
| temperature    | float    | No       | Overrides the default temperature                   |
| repeat_penalty | float    | No       | Overrides the default repeat penalty                |
| max_tokens     | int      | No       | Overrides the maximum generated token count         |


**Streaming response (SSE) format:**

```text
data: {"data": "你好", "done_flag": false}
data: {"data": "！", "done_flag": false}
data: {"data": "", "done_flag": true}
```


**Non-streaming response:**

```json
{"response": "你好！我是一个AI助手..."}
```


**Request example (curl):**

```bash
curl -X POST http://localhost:8050/api/chat \
  -H "Content-Type: application/json" \
  -d "{  \"message\": {    \"role\": \"user\",    \"content\": \"你好，请介绍一下自己\"  },  \"session_id\": 1,  \"stream\": true,  \"kb_ids\": null,  \"temperature\": null,  \"repeat_penalty\": null,  \"max_tokens\": null,  \"model_type\": \"llm\"}"
```

---

## Sessions

**Prefix:** `/api/sessions`

---

### POST `/api/sessions`
创建新会话，根据第一条消息自动生成会话名称（最多 50 字符）。

**请求体（JSON）：**
```json
{"first_message": "帮我写一首诗"}
```

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/sessions \
  -H "Content-Type: application/json" \
  -d "{\"first_message\": \"帮我写一首诗\"}"
```

**响应示例：**
```json
{"session_id": 1, "session_name": "帮我写一首诗"}
```

---

### GET `/api/sessions`
获取会话列表，按 `updated_at` 降序排列。

**查询参数：**
| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| limit | int | 50 | 返回数量上限 |
| offset | int | 0 | 偏移量 |

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/sessions
```

**响应示例：**
```json
{
  "sessions": [
    {
      "id": 1,
      "session_name": "帮我写一首诗",
      "created_at": "2026-03-24T10:00:00",
      "updated_at": "2026-03-24T10:05:00",
      "message_count": 4,
      "total_tokens": 512
    }
  ],
  "total": 1
}
```

---

### GET `/api/sessions/{session_id}`
获取指定会话的详情。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/sessions/1
```

**响应：** 同会话列表中单条数据格式。

**错误：** `404` 会话不存在

---

### PUT `/api/sessions/{session_id}/name`
更新会话名称。

**请求体（JSON）：**
```json
{"new_name": "新的会话名称"}
```

**请求示例 (curl)：**
```bash
curl -X PUT http://localhost:8050/api/sessions/1/name \
  -H "Content-Type: application/json" \
  -d "{\"new_name\": \"新的会话名称\"}"
```

**响应示例：**
```json
{"success": true, "message": "Session name updated successfully"}
```

---

### DELETE `/api/sessions/{session_id}`
删除会话（级联删除所有消息）。

**请求示例 (curl)：**
```bash
curl -X DELETE http://localhost:8050/api/sessions/1
```

**响应示例：**
```json
{"success": true, "message": "Session deleted successfully"}
```

---

### GET `/api/sessions/{session_id}/messages`
获取会话的所有消息。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/sessions/1/messages
```

**响应示例：**
```json
{
  "messages": [
    {
      "id": 1,
      "session_id": 1,
      "role": "user",
      "content": "帮我写一首诗",
      "token_count": 8,
      "created_at": "2026-03-24T10:00:00",
      "metadata": null
    }
  ],
  "session_id": 1,
  "total_tokens": 8
}
```

---

### DELETE `/api/sessions/{session_id}/messages`
清空会话中的所有消息（保留会话本身）。

**请求示例 (curl)：**
```bash
curl -X DELETE http://localhost:8050/api/sessions/1/messages
```

**响应示例：**
```json
{"success": true, "message": "Messages cleared successfully"}
```

---

## Knowledge Bases - 知识库管理

**前缀：** `/api/knowledge-bases`

---

### GET `/api/knowledge-bases`
获取所有知识库列表。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/knowledge-bases
```

**响应示例：**
```json
{
  "success": true,
  "knowledge_bases": [
    {
      "id": 1,
      "name": "技术文档库",
      "description": "存放技术相关文档",
      "avatar_url": "/api/assets/minio/avatars/kb_1.png",
      "file_count": 3,
      "created_at": "2026-03-24T10:00:00"
    }
  ],
  "count": 1
}
```

---

### POST `/api/knowledge-bases`
创建新知识库，支持上传头像。

**请求体（form-data）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 知识库名称 |
| description | string | 否 | 描述 |
| avatar | file | 否 | 头像图片（PNG/JPEG/GIF/WEBP，最大 5MB） |

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/knowledge-bases \
  -F "name=技术文档库" \
  -F "description=测试描述"
```

**响应示例：**
```json
{
  "success": true,
  "message": "Knowledge base '技术文档库' created successfully",
  "kb_id": 1,
  "knowledge_base": { "..." : "..." }
}
```

**错误：** `409` 知识库名称已存在

---

### GET `/api/knowledge-bases/{kb_id}`
获取指定知识库的详情。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/knowledge-bases/1
```

**响应示例：**
```json
{"success": true, "knowledge_base": { "..." : "..." }}
```

---

### PUT `/api/knowledge-bases/{kb_id}`
更新知识库信息（名称、描述、头像）。

**请求体（form-data）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 新名称 |
| description | string | 否 | 新描述 |
| avatar | file | 否 | 新头像（PNG/JPEG/GIF/WEBP，最大 5MB） |

**请求示例 (curl)：**
```bash
curl -X PUT http://localhost:8050/api/knowledge-bases/1 \
  -F "name=新名称"
```

**响应示例：**
```json
{"success": true, "message": "Knowledge base updated successfully", "knowledge_base": { "..." : "..." }}
```

---

### DELETE `/api/knowledge-bases/{kb_id}`
删除知识库（级联删除文件、向量数据、头像）。

**请求示例 (curl)：**
```bash
curl -X DELETE http://localhost:8050/api/knowledge-bases/1
```

**响应示例：**
```json
{"success": true, "message": "Knowledge base '技术文档库' deleted successfully"}
```

---

### POST `/api/knowledge-bases/{kb_id}/files`
上传单个文件到知识库（同步处理，完成后返回）。

**请求体（form-data）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| file | file | 是 | 要上传的文档文件 |

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/knowledge-bases/1/files \
  -F "file=@/path/to/document.pdf"
```

**响应示例：**
```json
{
  "success": true,
  "message": "File 'document.pdf' uploaded and processed successfully",
  "file_id": 42,
  "kb_id": 1
}
```

---

### POST `/api/knowledge-bases/{kb_id}/files/batch`
批量上传多个文件，异步处理（并发数上限为 3）。可使用进度流端点监控处理状态。

**请求体（form-data）：**
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| files | file[] | 是 | 多个文件 |

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/knowledge-bases/1/files/batch \
  -F "files=@/path/to/doc1.pdf" \
  -F "files=@/path/to/doc2.pdf"
```

**响应示例：**
```json
{
  "success": true,
  "files": [
    {"file_id": 42, "filename": "doc1.pdf", "status": "processing"},
    {"filename": "bad.xyz", "status": "error", "error": "不支持的文件类型: .xyz"}
  ],
  "message": "Uploaded 2 files"
}
```

---

### GET `/api/knowledge-bases/{kb_id}/files`
获取知识库中的所有文件列表。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/knowledge-bases/1/files
```

**响应示例：**
```json
{
  "success": true,
  "files": [
    {
      "id": 42,
      "filename": "document.pdf",
      "status": "completed",
      "created_at": "2026-03-24T10:00:00"
    }
  ],
  "count": 1
}
```

---

### DELETE `/api/knowledge-bases/{kb_id}/files/{file_id}`
从知识库中删除指定文件（同时删除 MinIO 存储、向量数据和数据库记录）。

**请求示例 (curl)：**
```bash
curl -X DELETE http://localhost:8050/api/knowledge-bases/1/files/42
```

**响应示例：**
```json
{"success": true, "message": "File 'document.pdf' deleted successfully"}
```

---

### GET `/api/knowledge-bases/{kb_id}/files/progress/stream` (SSE)
SSE 流：实时推送知识库中所有文件的处理进度（处理中 0.5s，空闲时 2s）。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/knowledge-bases/1/files/progress/stream
```

**SSE 事件格式：**
```
event: progress
data: {"files": [...], "timestamp": "2026-03-24T10:00:00"}
```

---

### GET `/api/knowledge-bases/{kb_id}/files/{file_id}/chunks`
获取文件在 ChromaDB 中的所有 chunk 信息（调试用途）。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/knowledge-bases/1/files/42/chunks
```

**响应示例：**
```json
{
  "kb_id": 1,
  "file_id": 42,
  "chunk_count": 15,
  "chunks": [
    {"document": "...", "metadata": {"source": "document.pdf", "chunk_index": 0}}
  ]
}
```

---

## Config - 配置管理

**前缀：** `/api/config`

---

### GET `/api/config/rag`
获取当前 RAG 配置（检索参数 + 提示词）。

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/config/rag
```

**响应示例：**
```json
{
  "retrieval": {
    "top_k": 3,
    "initial_k": 50,
    "intermediate_k": 10,
    "embed_weight": 0.5,
    "bm25_weight": 0.5,
    "rerank_enable": false
  },
  "prompts": {
    "conversation_system_prompt": "你是一个有用的助手...",
    "rag_system_prompt_template": "根据以下资料回答问题：\n\n{context}\n\n问题：{question}"
  }
}
```

---

### PUT `/api/config/rag`
更新 RAG 配置，保存到用户配置文件并立即热重载生效。

**请求体（JSON）：**
```json
{
  "retrieval": {
    "top_k": 5,
    "initial_k": 50,
    "intermediate_k": 10,
    "embed_weight": 0.6,
    "bm25_weight": 0.4,
    "rerank_enable": false
  },
  "prompts": {
    "conversation_system_prompt": "你是一个有用的助手...",
    "rag_system_prompt_template": "根据以下资料回答：\n\n{context}\n\n{question}"
  }
}
```

参数约束：
- `top_k`：1–20，且须 ≤ `intermediate_k`
- `intermediate_k`：5–50，且须 ≤ `initial_k`
- `initial_k`：10–200
- `embed_weight` / `bm25_weight`：0.0–1.0
- `rag_system_prompt_template` 必须包含 `{context}` 占位符

**请求示例 (curl)：**
```bash
curl -X PUT http://localhost:8050/api/config/rag \
  -H "Content-Type: application/json" \
  -d "{  \"retrieval\": {    \"top_k\": 5,    \"initial_k\": 50,    \"intermediate_k\": 10,    \"embed_weight\": 0.6,    \"bm25_weight\": 0.4,    \"rerank_enable\": false  },  \"prompts\": {    \"conversation_system_prompt\": \"你是一个有用的助手...\",    \"rag_system_prompt_template\": \"根据以下资料回答：\n\n{context}\n\n{question}\"  }}"
```

**响应示例：**
```json
{"message": "RAG配置已保存并立即生效"}
```

---

### POST `/api/config/rag/reset`
重置 RAG 配置为系统默认值，立即热重载生效。

**请求示例 (curl)：**
```bash
curl -X POST http://localhost:8050/api/config/rag/reset
```

**响应：** 返回重置后的完整 RAG 配置（同 `GET /api/config/rag` 格式）

---

## Assets - 静态资源

**前缀：** `/api/assets`

---

### GET `/api/assets/minio/{file_path}`
从 MinIO 代理提供静态文件（图片、PDF、文档等）。

**路径参数：**
| 参数 | 说明 |
|------|------|
| file_path | MinIO 中的文件路径，如 `kb_1/file_123.pdf` |

**特殊行为：**
- PPTX/PPT 文件会自动尝试提供对应的 PDF 预览版（`xxx_preview.pdf`），不存在时回退为原始文件

**支持的内容类型：**
- 图片：PNG、JPEG、GIF、WEBP、SVG
- 文档：PDF、TXT、MD
- Office：PPTX/PPT、DOCX/DOC、XLSX/XLS

**请求示例 (curl)：**
```bash
curl -X GET http://localhost:8050/api/assets/minio/kb_1/file_123.pdf
```

**响应：** 文件流（`StreamingResponse`），缓存 1 天

**错误：**
| 状态码 | 说明 |
|--------|------|
| 400 | 路径包含 `..` 或以 `/` 开头（目录穿越攻击防护） |
| 404 | 文件不存在 |
| 503 | MinIO 客户端未初始化 |
