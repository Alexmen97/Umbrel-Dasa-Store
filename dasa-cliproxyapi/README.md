# CLIProxyAPI for umbrelOS

[CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI) wraps Antigravity, ChatGPT Codex, Claude Code, Grok Build, Muse Code, and Devin as an OpenAI / Anthropic Claude / Google Gemini compatible API service.

## Configuration & Ports

- **Port**: `8317`
- **Web UI Management Center**: `http://<umbrel-ip>:8317/management.html`
- **Default Management Key**: `umbrel`
- **Default API Key**: `sk-cliproxy-umbrel`
- **Config file**: `data/config.yaml`

## Features

- **OpenAI Compatible**: `POST /v1/chat/completions`, `GET /v1/models`
- **Claude Compatible**: `POST /v1/messages`
- **Gemini Compatible**: `POST /v1beta/models/...`
- **Multi-architecture**: Native support for `linux/amd64` and `linux/arm64`.
