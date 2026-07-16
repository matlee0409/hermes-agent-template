# Hermes Agent — Admin Dashboard

A Python admin server (Starlette + Uvicorn) that manages [Hermes Agent](https://github.com/NousResearch/hermes-agent) — an autonomous AI agent by Nous Research — with a dark-themed web dashboard.

## How to Run

```bash
PORT=5000 python server.py
```

The server starts on port 5000. Log in with:
- **Username**: `admin` (or `ADMIN_USERNAME` env var)
- **Password**: auto-generated and printed to logs on first run (set `ADMIN_PASSWORD` env var to fix it)

## Architecture

```
Admin Server (Starlette/Uvicorn, port 5000)
├── /setup         — Admin dashboard (auth-gated)
├── /health        — Health check (public)
├── /login /logout — Cookie auth
└── /*             — Reverse proxy → hermes dashboard (port 9119)
```

The server manages two subprocesses:
- `hermes gateway` — the AI agent (started after configuration)
- `hermes dashboard` — the native Hermes web UI (always started)

Config is stored in `$HERMES_HOME/.env` and `$HERMES_HOME/config.yaml` (defaults to `~/.hermes/`).

## Dashboard Sections

| Section   | Description |
|-----------|-------------|
| Setup     | Configure LLM provider, messaging channels, tools |
| Status    | Gateway state, uptime, providers/channels overview |
| Logs      | Streaming gateway log viewer |
| **Terminal** | **Full interactive shell — run hermes commands, inspect files, etc.** |
| Users     | Approve/deny/revoke pairing requests |
| Backup    | GitHub-based data backup and restore |
| Update    | Check and apply Hermes Agent updates |

## Terminal Feature

The Terminal section opens a full PTY-backed bash shell via WebSocket (`/setup/api/terminal/ws`). It supports:
- Full interactive programs (hermes CLI, vim, htop, etc.)
- ANSI colors and escape sequences (xterm.js renderer)
- Dynamic resize tracking
- Reconnect button

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `PORT` | `8080` | Server port |
| `ADMIN_USERNAME` | `admin` | Login username |
| `ADMIN_PASSWORD` | *(auto-generated)* | Login password — printed to logs if unset |
| `HERMES_HOME` | `~/.hermes` | Path to Hermes data directory |

All other config (LLM provider, messaging channels, tool API keys) is set through the admin dashboard and stored in `$HERMES_HOME/.env`.

## User Preferences

- Keep the project's existing stack (Starlette + Jinja2 + Alpine.js + IBM Plex fonts)
- All admin routes are prefixed `/setup/` — keep public routes (`/health`, `/login`, `/logout`) clean
