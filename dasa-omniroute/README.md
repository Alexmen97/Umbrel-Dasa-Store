# OmniRoute for umbrelOS

[OmniRoute](https://github.com/ChrisCompton/omniroute) is a unified AI gateway and smart router for multi-provider LLMs. It exposes OpenAI, Anthropic Claude, and Google Gemini compatible endpoints with smart routing, automatic fallback, load balancing, and real-time latency monitoring.

## Configuration & Ports

- **Port**: `20128`
- **Dashboard Web UI**: `http://<umbrel-ip>:20128/`
- **OpenAI Compatible Endpoint**: `http://<umbrel-ip>:20128/v1/chat/completions`
- **Claude Compatible Endpoint**: `http://<umbrel-ip>:20128/v1/messages`
- **Persistent Data**: `${APP_DATA_DIR}/data` (SQLite database & auto-generated secrets)

## Features

- **Multi-Provider Support**: OpenAI, Anthropic Claude, Google Gemini, Ollama, OpenRouter, Groq, Mistral, DeepSeek, Cerebras, Cohere, and more.
- **Model Combos & Smart Fallback**: Automatically failover to backup or free models when primary limits are reached.
- **Next.js Web Dashboard**: Live latency (TTFT), token analytics, cost tracking, and provider health.
- **Zero-Config Bootstrap**: Automatically generates internal encryption keys and JWT secrets on first start.
- **Multi-Architecture**: Native support for `linux/amd64` and `linux/arm64`.
