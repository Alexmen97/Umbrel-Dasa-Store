# OpenBot Server for Umbrel (Dasa)

Official upstream image: `ghcr.io/nightly-labs/openbot:0.72.0@sha256:516e7cf80e606c8437b6f363c1eb7c399ca20a8cae157237cc0c941246c61db7`, pinned to its multi-architecture digest. Supports amd64 and arm64.

## What this package is

An always-on OpenBot server with a built-in guided setup page behind Umbrel authentication (port 8787). The official container serves the initial account setup page directly via browser. For everyday agent interaction, use the desktop app, iPhone app or https://openbot.run/app. Account and remote connection services remain external. This package does not deploy the separate self-hosted account stack.

## First login

Open the OpenBot Server app from your Umbrel dashboard. The built-in setup page guides you through signing into your OpenBot account: enter your email and the verification code sent to your inbox. Once connected, the page confirms that the server is online.

Alternatively, from the Umbrel terminal (Settings → Advanced Settings → Terminal → App → OpenBot Server, service `server`):

```sh
openbot login
openbot status
```

Or from the Umbrel host terminal:

```sh
docker exec -it dasa-openbot_server_1 openbot login
docker exec dasa-openbot_server_1 openbot status
```

Sign into the same account in the client, select the server, then configure a provider there. Provider usage may incur separate fees. For phone pairing run `openbot mobile-connect` inside server.

No real API keys or personal endpoints are included. Optional provider environment variables belong on service `server` through Umbrel Advanced App Settings. Use your own values privately; examples for public documentation are `tua_api` and `https://tuo_endpoint/v1`. Not every provider uses environment variables; follow upstream instructions.

## Runtime and security

- Official seccomp profile from **v0.52.0**, kept alongside the manifest. Allows user namespaces for Electron. Never replace with privileged, seccomp=unconfined or --no-sandbox.
- UID/GID 1000:1000; no-new-privileges; 2 GiB shared memory; 8 GiB server memory limit; 60-second graceful shutdown.
- No server ports published. The built-in setup page is served behind Umbrel app_proxy on port 8080 with normal proxy authentication.
- `hooks/pre-start` creates the private data mount owned by UID/GID 1000. It never recursively chowns agent files.
- Agents run commands without per-action approvals and can change all their data. No host home, other apps or Docker socket mounted. Container isolation does not block LAN access. Use network firewall rules as appropriate.
- Upstream labels this a development preview. Computer Use, remote desktop, Gemini login and restricted-workspace sandboxing have documented container limitations. arm64 has no remote desktop runtime.

## Persistence and upgrades

`${APP_DATA_DIR}/data/openbot` maps to `/data`, containing conversations, workspaces, downloaded tools, provider sessions, keyring and keyring password. Keep it all together, including hidden files. Stop the app before copying the complete directory. Protect backups as credentials. Do not run two instances on the same data or identity.

Updates are controlled by the Umbrel store, not an unattended image watcher. Before updating, review upstream compose and seccomp changes and create a complete stopped backup. Never roll an older image back against a database migrated by a newer image: recover the matching complete backup instead.

The built-in setup page is served directly by the OpenBot container when `OPENBOT_SETUP_PAGE_PORT` is set. Check `openbot status` and the server logs.

## Verification

The repository workflow validates the compose configuration (using an app_proxy stub), runs the real image with the shipped seccomp profile on an ephemeral GitHub runner, checks `openbot status`, restarts it on the same data and checks clean shutdown. It also tests the setup page endpoint. Account login and actual provider turns require user interaction and are not covered by the unauthenticated smoke test. Native umbrelOS installation remains a separate acceptance test.

## License and required notices

OpenBot uses [PolyForm Noncommercial License 1.0.0](https://polyformproject.org/licenses/noncommercial/1.0.0/). See [upstream license](https://github.com/nightly-labs/openbot/blob/v0.52.0/LICENSE) and [NOTICE](https://github.com/nightly-labs/openbot/blob/v0.52.0/NOTICE). Commercial use requires review/permission from upstream. The package references upstream images; it does not relicense the application.

Required Notice: Copyright 2026 Synthetify Labs spółka z ograniczoną responsabilità.

Required Notice: For a commercial license, contact licenses@openbot.run.
