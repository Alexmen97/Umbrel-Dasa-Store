# OpenBot Server for Umbrel (Dasa)

Official upstream image: `ghcr.io/nightly-labs/openbot:0.51.1`, pinned to its multi-architecture digest. Supports amd64 and arm64.

## What this package is

An always-on OpenBot server plus a static, local setup page behind Umbrel authentication (port 8787). **The official container does not serve the OpenBot web client.** Use the desktop app, iPhone app or https://openbot.run/app. Account and remote connection services remain external. This package does not deploy the separate self-hosted account stack.

## First login

Umbrel Settings → Advanced Settings → Terminal → App → OpenBot Server, then select the **server** container and run `openbot login`. Enter the email and one-time code yourself in that terminal. If the terminal opens the web container instead, select server; it is not a shell for running OpenBot commands.

Alternatively, from the Umbrel **host** terminal:

```sh
docker exec -it dasa-openbot_server_1 openbot login
docker exec dasa-openbot_server_1 openbot status
```

Sign into the same account in the client, select the server, then configure a provider there. Provider usage may incur separate fees. For phone pairing run `openbot mobile-connect` inside server.

No real API keys or personal endpoints are included. Optional provider environment variables belong on service `server` through Umbrel Advanced App Settings. Use your own values privately; examples for public documentation are `tua_api` and `https://tuo_endpoint/v1`. Not every provider uses environment variables; follow upstream instructions.

## Runtime and security

- Official seccomp profile from **v0.51.1**, kept alongside the manifest. Allows user namespaces for Electron. Never replace with privileged, seccomp=unconfined or --no-sandbox.
- UID/GID 1000:1000; no-new-privileges; 2 GiB shared memory; 8 GiB server memory limit; 60-second graceful shutdown.
- No server ports published. Only the static setup page is behind Umbrel app_proxy; normal proxy authentication is retained.
- `hooks/pre-start` creates the private data mount owned by UID/GID 1000. It never recursively chowns agent files.
- Agents run commands without per-action approvals and can change all their data. No host home, other apps or Docker socket mounted. Container isolation does not block LAN access. Use network firewall rules as appropriate.
- Upstream labels this a development preview. Computer Use, remote desktop, Gemini login and restricted-workspace sandboxing have documented container limitations. arm64 has no remote desktop runtime.

## Persistence and upgrades

`${APP_DATA_DIR}/data/openbot` maps to `/data`, containing conversations, workspaces, downloaded tools, provider sessions, keyring and keyring password. Keep it all together, including hidden files. Stop the app before copying the complete directory. Protect backups as credentials. Do not run two instances on the same data or identity.

Updates are controlled by the Umbrel store, not an unattended image watcher. Before updating, review upstream compose and seccomp changes and create a complete stopped backup. Never roll an older image back against a database migrated by a newer image: recover the matching complete backup instead.

The local setup page is deliberately static: its availability does not imply that the server is healthy or logged in. Check `openbot status` and the server logs.

## Verification

The repository workflow validates the compose configuration (using an app_proxy stub), runs the real image with the shipped seccomp profile on an ephemeral GitHub runner, checks `openbot status`, restarts it on the same data and checks clean shutdown. It also tests the static page. Account login and actual provider turns require user interaction and are not covered by the unauthenticated smoke test. Native umbrelOS installation remains a separate acceptance test.

## License and required notices

OpenBot uses [PolyForm Noncommercial License 1.0.0](https://polyformproject.org/licenses/noncommercial/1.0.0/). See [upstream license](https://github.com/nightly-labs/openbot/blob/v0.51.1/LICENSE) and [NOTICE](https://github.com/nightly-labs/openbot/blob/v0.51.1/NOTICE). Commercial use requires review/permission from upstream. The package references upstream images; it does not relicense the application.

Required Notice: Copyright 2026 Synthetify Labs spółka z ograniczoną odpowiedzialnością.

Required Notice: For a commercial license, contact licenses@openbot.run.
