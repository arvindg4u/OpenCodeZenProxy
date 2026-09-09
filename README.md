# OpenCodeZenProxy

A lightweight, memory-efficient FastAPI proxy that forwards all requests from `/v1/*` to `https://opencode.ai/zen/v1/*` verbatim — headers, query params, body, and method — and streams the response back.

## Features

- **Zero-copy streaming** — request body read via `request.stream()`, response forwarded via `resp.aiter_raw()`; no full buffering at any point
- **Transparent** — preserves `content-encoding`, passes through every header except hop-by-hop ones (`host`, `transfer-encoding`, `connection`, etc.)
- **Connection pooling** — shared `httpx.AsyncClient` with configurable pool limits
- **Railway-ready** — `railway.json` with health check, `$PORT` binding, and nixpacks build
- **Render-ready** — `render.yaml` blueprint with health check and `$PORT` binding

## Usage

```bash
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 1253
```

Then point your client at `http://localhost:1253/v1/...` instead of `https://opencode.ai/zen/v1/...`.

## Deploy to Railway

Click the button or use the CLI:

```bash
railway up
```

The `railway.json` handles build and start command automatically.

## Deploy to Render

Two options — pick whichever you prefer:

### Option A: Blueprint (recommended)

The repo already includes a `render.yaml` blueprint:

1. Push this repo to GitHub/GitLab
2. On [dashboard.render.com](https://dashboard.render.com) → **New** → **Blueprint**
3. Connect your repo → Render reads `render.yaml` and creates the service automatically

### Option B: Manual web service

1. **New** → **Web Service** → connect your repo
2. Settings:
   - **Runtime:** Python
   - **Build Command:** `pip install -r requirements.txt`
   - **Start Command:** `uvicorn main:app --host 0.0.0.0 --port $PORT`
   - **Health Check Path:** `/health`
3. Click **Create Web Service**

Optional environment variables (Settings → Environment):

| Variable | Default | Purpose |
|----------|---------|---------|
| `UPSTREAM_BASE` | `https://opencode.ai/zen/v1` | Override where `/v1/*` requests are forwarded |
| `PYTHON_VERSION` | `3.12.0` | Pin the Python runtime version |

> **Free tier note:** Render free instances spin down after ~15 min of inactivity.
> The first request after a cold start can take up to a minute while the instance
> wakes up; subsequent requests are fast.

## Endpoints

| Path | Description |
|------|-------------|
| `/v1/{path}` | Proxies to `https://opencode.ai/zen/v1/{path}` |
| `/health` | Health check (used by Railway) |

## License

MIT
