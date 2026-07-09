# Hermes Agent — Railway Template

## Overview
Imported from GitHub: deploys [Hermes Agent](https://github.com/NousResearch/hermes-agent) with a
Python (Starlette + Uvicorn) admin dashboard for configuration, gateway management, and user pairing.
Designed to run on Railway (Dockerfile + persistent `/data` volume), started via `start.sh` → `server.py`.

- `server.py` — admin server: setup wizard (`/setup`), management API (`/setup/api/*`), reverse
  proxy to the native Hermes dashboard, and the cookie-based login gate.
- `templates/index.html` — the single-page admin UI (Alpine.js).
- `Dockerfile` — pins the installed Hermes Agent version via `ARG HERMES_REF` (bump this + redeploy
  to update; check https://github.com/NousResearch/hermes-agent/releases for the latest tag).

## GitHub Data Backup
Added a "Backup" section (Setup UI) that commits/pushes `$HERMES_HOME` (config, workspace, memories,
pairing state) to a GitHub repo the user owns — new or existing — so data survives volume loss or
redeploys. Modeled after the `alphaclaw` template's git-sync approach.

- Config: `GITHUB_BACKUP_TOKEN`, `GITHUB_BACKUP_REPO` (`owner/name`), `GITHUB_BACKUP_MODE`
  (`new`/`existing`), `GITHUB_BACKUP_AUTO` (hourly auto-sync toggle) — stored like any other env var.
- Endpoints: `POST /setup/api/backup/verify`, `POST /setup/api/backup/sync`, `GET /setup/api/backup/status`.
- Secrets are never committed: `.env`, `auth.json`, and PID/socket files are excluded via a managed
  `.gitignore` in `$HERMES_HOME` before every `git add -A`.
- Sync targets the *remote's* actual default branch (queried via `git ls-remote --symref`), not
  whatever a fresh local `git init` happens to name `HEAD` — otherwise syncing an existing repo would
  silently create a second, disconnected branch.

## User preferences
None recorded yet.
