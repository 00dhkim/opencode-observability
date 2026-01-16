# OpenCode Observability Hands On

[한국어 README](README.md)

This is a hands-on project to understand the internal workings of OpenCode using LiteLLM, Phoenix, and OpenCode.
In a local development environment, OpenCode requests are sent to a LiteLLM Proxy, which then routes them to actual LLM providers (Google Gemini, Grok, etc.). All request/response logs are observed through Phoenix.

## 🚀 Components

- **LiteLLM Proxy**: An OpenAI-compatible proxy that receives client requests and delivers them to the appropriate LLM models.
- **Phoenix**: A UI tool for LLM execution tracking and observability.
- **Postgres**: Stores metadata for the proxy.
- **OpenCode**: An AI coding agent CLI tool that users interact with.

## 📁 Key Files

- `docker-compose.yml`: Runs Proxy, Phoenix, and Postgres containers.
- `litellm_config.yaml`: Defines model aliases and actual model ID mappings.
- `.env`: Configuration file for API keys and proxy settings.
- `opencode.json`: OpenCode configuration file containing local proxy (`dohyun_litellm`) connection info. (Can be copied to `~/.config/opencode/opencode.json` for global configuration.)

## ▶️ How to Run

### 1) Set API Keys

- Copy `.env.example` to create a `.env` file.
- To use the Grok Code Fast 1 model for free, you must obtain a key from [https://opencode.ai/zen](https://opencode.ai/zen) and enter it into `OPENCODE_ZEN_API_KEY`.
- (Optional) Enter the key obtained from Google AI Studio ([https://aistudio.google.com/api-keys](https://aistudio.google.com/api-keys)) into `GEMINI_API_KEY`.

### 2) Start Containers

```bash
docker compose up -d
```

### 3) Run OpenCode and Select Model

```bash
opencode
```

- Select the configured model provider (`litellm by dohyun (local)`) in the OpenCode TUI.

## 🔎 Observability

Access the Phoenix UI to check real-time LLM request/response logs:
👉 [http://localhost:6006](http://localhost:6006)

## 🧪 Direct Proxy Call Tests

### 1) View Model List

Check the list of all models available on the current proxy.

```bash
curl -s -H "Authorization: Bearer sk-1234567890" http://localhost:4000/v1/models | jq
```

### 2) Gemini Model Test (Google SDK/API Style)

Example of calling the `gemini-2.5-flash-lite` model using the Google AI style endpoint.

```bash
curl -X POST "http://localhost:4000/v1beta/models/gemini-2.5-flash-lite:generateContent" \
  -H "Authorization: Bearer sk-1234567890" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [
      { "parts": [{ "text": "Hi, who are you?" }], "role": "user" }
    ]
  }' | jq
```

### 3) Grok Code Model Test (OpenAI Style)

Example of calling the `grok-code` model using the OpenAI-compatible endpoint (`/v1/chat/completions`).

```bash
curl -X POST "http://localhost:4000/v1/chat/completions" \
  -H "Authorization: Bearer sk-1234567890" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "grok-code",
    "messages": [
      { "role": "user", "content": "Write a simple Python code." }
    ]
  }' | jq
```

## ❗ Troubleshooting

- Ensure the `baseURL` in `opencode.json` matches the proxy address (`http://localhost:4000/v1`).
- Verify that valid API keys are set in the `.env` file.
- Check LiteLLM logs:
  ```bash
  docker logs -f litellm
  ```
