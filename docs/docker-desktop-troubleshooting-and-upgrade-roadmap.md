# Docker Desktop Troubleshooting + Upgrade Roadmap (Windows)

## Why the current error happens

If `docker version` fails with:

`open //./pipe/dockerDesktopLinuxEngine: The system cannot find the file specified`

then the Docker CLI is configured to use the **Docker Desktop Linux engine named pipe**, but the engine is not currently available. This almost always means Docker Desktop (or its Linux backend) is not running correctly.

Your `docker context ls` output shows `desktop-linux` is selected, and your `docker version` output confirms the client tries this endpoint:

- `npipe:////./pipe/dockerDesktopLinuxEngine`

If that pipe is missing, the daemon cannot be reached and any `docker compose up` operation fails before image pull.

## Immediate recovery checklist

1. Start (or restart) Docker Desktop.
2. Ensure Docker Desktop is set to **Linux containers** mode.
3. Re-run:
   - `docker context use desktop-linux`
   - `docker version`
4. If still failing:
   - Docker Desktop → Troubleshoot → **Restart Docker Desktop**.
   - If needed, Docker Desktop → Troubleshoot → **Clean / Purge data** (last resort).
5. Verify WSL is healthy from PowerShell:
   - `wsl --status`
   - `wsl -l -v`
6. If WSL commands are blank/hung, repair WSL then restart Windows.

## Compose modernization tasks (safe, low risk)

- Remove obsolete top-level `version` key from Compose files (Compose v2 ignores it).
- Keep image tags explicit and consistent across environments.
- Add healthchecks for critical services (`db`, `wiki`).
- Add a `.env.example` with expected variables.

## Postgres upgrade roadmap (15 -> 17)

### Phase 0: Preparation
- Inventory current DB version and data size.
- Confirm Wiki.js version compatibility with Postgres 17.
- Freeze schema changes for migration window.

### Phase 1: Backups and rollback
- Create full logical backup (`pg_dumpall` or per-db dumps).
- Validate backup restoration into a throwaway Postgres container.
- Write a rollback runbook with explicit stop/repoint steps.

### Phase 2: Staging rehearsal
- Clone production-like data into staging.
- Run app against Postgres 17 in staging.
- Execute smoke tests: login, page read/write, search, attachments.

### Phase 3: Production cutover
- Put app in maintenance mode / reduce writes.
- Final incremental backup.
- Promote Postgres 17 stack.
- Run DB migrations and health checks.
- Monitor logs and query latency.

### Phase 4: Post-cutover hardening
- Keep old 15 data snapshot for agreed retention window.
- Enable regular automated backup verification.
- Document new baseline versions and operational SOPs.

## Questions to answer before finalizing the plan

1. Is this local-only, or do you also run a production deployment?
2. Are you using Docker Desktop + WSL2 backend on this machine?
3. Which Windows version/build are you on?
4. Do you want to standardize all local Compose files on Postgres 15 first, then upgrade to 17 in a separate change?
5. Do you already have a backup/restore script we should align with?
