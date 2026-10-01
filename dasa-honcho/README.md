# Honcho + OpenConcho on Umbrel

This package includes Honcho **3.2.2** and OpenConcho **0.16.2**, both pinned by image digest for Linux AMD64 and ARM64.

Open Honcho from the Umbrel dashboard to launch OpenConcho. The first browser visit automatically connects to the bundled API at `http://api:8000`. This is a Docker-internal address: OpenConcho proxies requests server-side, so the browser does not need Docker DNS or CORS configuration.

## Addresses

- Dashboard: `http://<umbrel-host>:8000/`
- Honcho API: `http://<umbrel-host>:8000/v3/...`
- API documentation: `http://<umbrel-host>:8000/docs`
- Honcho health: `http://<umbrel-host>:8000/health`

Existing clients can retain `http://<umbrel-host>:8000` as the Honcho base URL. Inside the stack, services use `http://api:8000`.

The dashboard's upstream allowlist permits only `api`. Adding arbitrary remote instances in OpenConcho Settings is therefore not supported by this package's default proxy configuration.

## Data and settings

Honcho retains its existing PostgreSQL, Redis, and app-data mounts. OpenConcho stores connection preferences in the browser, so it does not need a server-side data volume. Preferences are specific to that browser and origin; clearing browser storage removes them, but does not remove Honcho memories.

Provider credentials and model configuration must be supplied to both Honcho API and deriver using the [Honcho v3 configuration reference](https://github.com/plastic-labs/honcho/blob/v3.2.2/config.toml.example). They are needed for AI actions; this store does not include API keys or private provider endpoints.

The existing package runs with Honcho authentication disabled. Restrict access to a trusted network or configure authentication before exposing it publicly.

## Upgrades and verification

Store revision `3.2.2-1` adds the dashboard without changing the Honcho backend version. Back up PostgreSQL before upgrading from Honcho 2.x; the API entrypoint provisions/migrates the database before starting and OpenConcho waits for API health.

Verify on Umbrel: open the dashboard, list/create a workspace, confirm `/health` returns JSON and `/docs` opens Swagger, then check an existing API client. AI chat additionally requires valid provider configuration. Container launch, data migration, and browser behavior require runtime validation on the actual Umbrel device.
