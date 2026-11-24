# **Trae Local API**

Wrap Trae IDE as a local OpenAI / Anthropic-compatible API service, allowing AI Agent tools such as Claude Code, Cursor, and Cline to directly call Trae's underlying models (GLM-5.2, DeepSeek, Qwen, etc.).

## **Features**

- **OpenAI-compatible API** — `/v1/chat/completions`, usable directly with the OpenAI SDK.
- **Anthropic-compatible API** — `/v1/messages`, supporting multi-turn tool calls in tools such as Claude Code.
- **Streaming** — SSE streaming responses that return model output in real time.
- **Reasoning support** — Supports `reasoning_content` (chain of thought), compatible with reasoning models such as DeepSeek.
- **Automatic authentication** — Automatically extracts JWT tokens from Trae IDE storage and supports automatic token refresh.
- **CN edition decryption** — Decrypts Trae CN's custom "tc" encryption format (AES-128-CBC + SHA-512).
- **Multi-version support** — Supports both CN (China edition) and SG (international edition).
- **5 model tiers** — Models are classified into 5 capability tiers, with concurrent racing within the same tier and fallback across tiers.
- **Smart fallback** — Automatically switches to a model in the same tier or the next tier when queue times are too long.
- **Automatic multimodal switching** — Automatically switches to an image-capable model when image input is detected.
- **Proxy support** — HTTP/SOCKS5 proxies.
- **File output** — AI responses can be saved directly to files on disk.
- **Traffic logging** — Full request/response logging for debugging.

## **Getting Started for New Users**

### **Step 1: Install Trae IDE**

1. Download and install [Trae IDE](https://trae.cn/?utm_source=chatgpt.com) (users in China should choose the CN edition).
2. Open Trae IDE and sign in to your account (WeChat/mobile phone login is supported).
3. Make sure AI chat works properly in Trae IDE.

> **Important:** You must have signed in to Trae IDE at least once. This service needs to read authentication information from Trae's local storage.

### **Step 2: Install Node.js**

- Install Node.js >= 18 (20+ recommended).
- Verify the installation: `node --version`

### **Step 3: Clone and Install**

```
git clone https://github.com/<your-username>/trae-local-api.git
cd trae-local-api
npm install
```

### **Step 4: Configure Environment Variables**

```
cp .env.example .env
```

Edit the `.env` file:

```
# Trae edition: cn (China edition) or sg (international edition)
TRAE_EDITION=cn

# Local API key (required for client access; customize as desired)
API_KEY=trae-local-api-key

# Server port
PORT=19900

# Default workspace directory
WORKSPACE_DIR=./output
```

### **Step 5: Start the Service**

```
npm start
```

The following logs indicate that the service has started successfully:

```
[model-config] Loaded 37 model mappings from model-config.json
[auth] Using CN edition auth data (decrypted)
[server] Trae Local API running on http://localhost:19900
```

### **Step 6: Verify the Service**

```
# Check service status
curl http://localhost:19900/v1/status -H "Authorization: Bearer trae-local-api-key"

# List available models
curl http://localhost:19900/v1/models -H "Authorization: Bearer trae-local-api-key"

# Test a simple conversation
curl -N http://localhost:19900/v1/chat/completions \
  -H "Authorization: Bearer trae-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{"model":"auto","messages":[{"role":"user","content":"Hello!"}],"stream":true}'
```

### **Step 7: Configure Claude Code**

```
# Using environment variables in PowerShell
$env:ANTHROPIC_BASE_URL = "http://localhost:19900"
$env:ANTHROPIC_API_KEY = "trae-local-api-key"
claude
```

Or create `.claude/settings.json`:

```
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:19900",
    "ANTHROPIC_API_KEY": "trae-local-api-key"
  }
}
```

### **Common Setup Issues**

#### **Q: Startup fails with "No readable auth info found"**

**Cause:** Trae IDE is not signed in, or the authentication storage path is incorrect.

**Solution:**

1. Make sure Trae IDE (CN edition) is installed and you are signed in.
2. Check whether Trae IDE's storage file exists:
   - CN edition: `%APPDATA%\Trae-CN\User\globalStorage\state.vscdb`
   - SG edition: `%APPDATA%\Trae\User\globalStorage\state.vscdb`
3. If the file exists but the error persists, set the `TRAE_MANUAL_TOKEN` environment variable:
   - Open Trae IDE → F12 Developer Tools → Network.
   - Trigger an AI conversation and find the request header containing `Authorization: Cloud-IDE-JWT eyJ...`.
   - Copy the `eyJ...` portion and set it in `.env`: `TRAE_MANUAL_TOKEN=eyJ...`

#### **Q: Token decryption fails in the CN edition**

**Cause:** Trae CN uses a custom "tc" encryption format that requires the correct decryption key.

**Solution:**

1. Make sure you are using the latest version of Trae CN IDE.
2. If decryption still fails, use the `TRAE_MANUAL_TOKEN` method.

#### **Q: Requests return a 401 authentication error**

**Cause:** The token has expired.

**Solution:**

1. Reopen Trae IDE and trigger a conversation to refresh the token.
2. Restart the trae-local-api service.

## **API Endpoints**

| Endpoint                 | Method | Purpose                                                        | Authentication |
| ------------------------ | ------ | -------------------------------------------------------------- | -------------- |
| `/`                      | GET    | Dashboard HTML page (traffic log viewer)                       | None           |
| `/health`                | GET    | Health check (returns status and uptime)                       | None           |
| `/v1/info`               | GET    | API endpoint catalog (descriptions of all available endpoints) | Required       |
| `/v1/status`             | GET    | Service status                                                 | Required       |
| `/v1/models`             | GET    | List available models                                          | Required       |
| `/v1/models/detail`      | GET    | Model details, including capabilities and tiers                | Required       |
| `/v1/chat/completions`   | POST   | OpenAI-compatible chat endpoint                                | Required       |
| `/v1/messages`           | POST   | Anthropic-compatible chat endpoint                             | Required       |
| `/v1/chat/file`          | POST   | Chat endpoint that saves responses to files                    | Required       |
| `/v1/dashboard/status`   | GET    | Dashboard status data                                          | Required       |
| `/v1/dashboard/sessions` | GET    | Dashboard session list                                         | Required       |
| `/v1/dashboard/requests` | GET    | Dashboard request records                                      | Required       |
| `/v1/dashboard/stats`    | GET    | Dashboard statistics                                           | Required       |

## **Usage**

### **cURL**

```
curl -N http://localhost:19900/v1/chat/completions \
  -H "Authorization: Bearer trae-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "auto",
    "messages": [{"role": "user", "content": "Hello!"}],
    "stream": true
  }'
```

### **Python (OpenAI SDK)**

```
from openai import OpenAIclient = OpenAI(    base_url="http://localhost:19900/v1",    api_key="trae-local-api-key")response = client.chat.completions.create(    model="auto",    messages=[{"role": "user", "content": "Hello!"}],    stream=True)for chunk in response:    if chunk.choices[0].delta.content:        print(chunk.choices[0].delta.content, end="", flush=True)
```

### **File Output**

```
curl -s http://localhost:19900/v1/chat/file \
  -H "Authorization: Bearer trae-local-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "auto",
    "messages": [{"role": "user", "content": "Write a Python HTTP server"}],
    "filename": "server.py",
    "overwrite": true
  }'
```

## **Agent Tool Configuration**

### **Claude Code**

```
$env:ANTHROPIC_BASE_URL = "http://localhost:19900"
$env:ANTHROPIC_API_KEY = "trae-local-api-key"
claude
```

### **Cursor**

- **API Key:** `trae-local-api-key`
- **Base URL:** `http://localhost:19900/v1`
- **Model:** `auto`

### **Cline (VS Code Extension)**

- API Provider: "OpenAI Compatible"
- Base URL: `http://localhost:19900/v1`
- API Key: `trae-local-api-key`
- Model: `auto`

## **Available Models**

### **Model Tiers (5 Tiers)**

| Tier | Name        | Models                                                            | Images | Reasoning |
| ---- | ----------- | ----------------------------------------------------------------- | ------ | --------- |
| T1   | Flagship    | glm-5.2                                                           | ✗      | ✓         |
| T2   | Powerful    | glm-5.1, qwen-3.7-plus, kimi-k2.6, DeepSeek-V4-Pro                | ✗/✓    | ✓         |
| T3   | Mid-range   | glm-5, qwen-3.6-plus, minimax-m3, DeepSeek-V4-Flash, glm-5v-turbo | ✗/✓    | ✓         |
| T4   | Lightweight | glm-4.7, kimi-k2, qwen3-coder, minimax-m2.7, Doubao-Seed-2.0-Code | ✗/✓    | ✓         |
| T5   | Lightest    | glm-4.6, Doubao_1_6, minimax-m2.1, minimax-m2                     | ✗/✓    | ✓         |

### **Claude Model Mapping**

| Claude Model            | Mapped To | Tier |
| ----------------------- | --------- | ---- |
| claude-opus-4-7/4-6/4-5 | glm-5.2   | T1   |
| claude-sonnet-4-6/4-5/4 | glm-5.2   | T1   |
| claude-3.5/3.7-sonnet   | glm-5.2   | T1   |
| claude-haiku-4-5        | glm-5.1   | T2   |

> Some models may require a paid Trae subscription. Both DeepSeek-V4-Pro and DeepSeek-V4-Flash support `tool_use`.

## **Smart Fallback**

### **How It Works**

```
User request → Primary model queue > threshold
  ↓
Race concurrently within the same tier (raceWithinTier)
  ↓ All models are queued
Fallback to the next tier (tieredFallback)
  ↓ All models are queued
Fallback model (fallbackModel: glm-5)
  ↓ Still queued
Continue waiting for the original model
```

### **Configuration File**

The `fallback` section in `model-config.json`:

```
{
  "fallback": {
    "autoFallback": true,
    "queueThreshold": 300,
    "tieredFallback": true,
    "raceWithinTier": true,
    "fallbackModel": "glm-5",
    "mappings": {
      "claude-sonnet-4-6": ["glm-5.1", "glm-5", "qwen-3.6-plus"]
    }
  }
}
```

### **Parameter Descriptions**

| Parameter        | Description                                                                  |
| ---------------- | ---------------------------------------------------------------------------- |
| `autoFallback`   | Whether automatic fallback is enabled (default: true)                        |
| `queueThreshold` | Queue position threshold; fallback is triggered when exceeded (default: 300) |
| `tieredFallback` | Whether tier-based fallback is enabled (default: true)                       |
| `raceWithinTier` | Whether models within the same tier are raced concurrently (default: true)   |
| `fallbackModel`  | Fallback model used when all tiers are queued (default: glm-5)               |
| `mappings`       | Legacy fallback chain configuration (backward compatible)                    |

### **Fallback Process**

1. **Detect queueing:** Trae returns a `request_wait_in_queue` event containing the queue position.
2. **Race within the same tier:** If `position > queueThreshold`, send concurrent requests to other models in the same tier; use the first response received.
3. **Fallback to the next tier:** If all models in the current tier are queued, fall back to the first model in the next tier.
4. **Fallback model:** If all tiers are queued, use `fallbackModel`.
5. **Continue waiting:** If the fallback model is also queued, continue waiting for the original model.

## **Multimodal Support**

When image content is detected in a message, the service automatically switches to an image-capable model in the same tier or the nearest tier.

Supported image-input models include: Doubao-Seed-2.0-Code, kimi-k2.6, qwen-3.7-plus, qwen-3.6-plus, minimax-m3, glm-5v-turbo, Doubao_1_6, and others.

## **Authentication**

### **Automatic Authentication**

The server automatically reads authentication information from Trae IDE storage:

- **SG edition:** Reads plaintext JSON tokens directly.
- **CN edition:** Automatically decrypts the "tc" encrypted format (AES-128-CBC + SHA-512).

### **Token Refresh**

- JWT tokens are valid for approximately 14 days; the server automatically detects and refreshes them.
- Refresh uses a mutex to prevent concurrent race conditions.
- If refresh fails, reopen Trae IDE and sign in again.

### **Manual Token (Fallback)**

Set `TRAE_MANUAL_TOKEN=eyJ...` in `.env`. The server will fall back to this token if automatic decryption fails.

## **Project Structure**

```
trae-local-api/
├── src/
│   ├── server.js           # Express server (OpenAI + Anthropic-compatible endpoints)
│   ├── trae-client.js      # Trae API client + model mapping + tiers + fallback
│   ├── auth.js             # Authentication management (automatic IDE version detection, token refresh mutex)
│   ├── anthropic-format.js # Anthropic format conversion (6 tool-call parsing formats)
│   ├── openai-format.js    # OpenAI format conversion
│   ├── trae-decrypt.js     # CN edition encryption-format decryption
│   ├── traffic-logger.js   # Request/response logging (with memory-leak protection)
│   ├── crypto.js           # AES-256-GCM encryption/decryption
│   └── uuid.js             # UUID generation
├── model-config.json       # Model mappings + tiers + fallback configuration
├── model-fallback.json     # Legacy fallback-chain configuration (backward compatible)
├── .env.example            # Environment variable template
├── .history.md             # Project history
├── package.json
└── README.md
```

## **FAQ**

### **Q: Claude Code only interacts for one turn and then stops**

**Fixed.** Causes:

1. XML-formatted tool calls from the model were not being parsed — now supports 6 formats.
2. Queue information polluted the conversation history — now only sends a ping.
3. Claude Code timed out while queued — now sends `message_start` earlier.

### **Q: Queue information pollutes the conversation history**

**Fixed.** Queue events now only send a ping (ignored by Claude Code), rather than being sent as text content.

### **Q: How do I view request logs?**

Logs are saved in the `logs/YYYY-MM-DD/workspace/` directory, with one JSON file per request. Open the Dashboard:

```
http://localhost:19900/
```

### **Q: How do I update the model list?**

After Trae CN updates its models, call the API to refresh:

```
curl http://localhost:19900/v1/models/detail -H "Authorization: Bearer trae-local-api-key"
```

Then manually update the model mappings and tiers in `model-config.json`.

## **License**

This project is intended for learning and research purposes only.
