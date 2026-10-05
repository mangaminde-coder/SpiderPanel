# SpiderPanel — Base44 Dev Environment

## What this is
A single-file Python FastAPI app (`main.py`, ~600KB) — an Xray/proxy control panel
for subscriptions, nodes, scanners, and Telegram automation. Serves a static SPA
from `static/` (login.html, index.html, sub.html).

## How it runs
- **Entrypoint:** `python -m uvicorn main:app --host 0.0.0.0 --port 8080 --reload`
- **Port:** always 8080 internally; mapped to host port 3000 in compose.
- **Persistence:** JSON file at `DATA_DIR/spider_state.json` (no database). `DATA_DIR` defaults to `/data` (a Docker volume).
- **No external credentials required to boot.** `ADMIN_PASSWORD` (default `admin`) and `SECRET_KEY` (default built-in) are optional env vars.

## Key routes
- `GET /` — JSON service info
- `GET /healthz` — health check (used by compose healthcheck)
- `GET /login` — login page (static HTML)
- `GET /spider` — main panel (auth-gated, redirects to /login if no session)
- `GET /sub/{uuid}` — public subscription page

## Startup behavior
- Downloads the Xray binary from GitHub on first boot (network required; has fallbacks if it fails).
- Discovers its public endpoint from request headers / platform env vars.
- Auto-creates default inbounds (TLS+WS, Reality+XHTTP, Node selector).
- MTProxy binary is NOT built in the dev image — the app gracefully handles its absence.

## Dev compose
`docker-compose.base44.yml` uses `python:3.13-slim`, bind-mounts the source at `/app`,
installs `requirements.txt` on startup, and runs uvicorn with `--reload`. State persists
in the `spider_data` volume at `/data`.

## Verify it works
```bash
curl http://localhost:3000/healthz   # → {"ok": true, ...}
curl http://localhost:3000/login     # → 200, login.html
```
Default login: password `admin`.
