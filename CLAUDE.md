# Workoflow Metrics

Grafana dashboards for monitoring the Workoflow platform (LLM costs, n8n workflows, tool usage, user activity).

## Deployment

### Production Server
- **Host:** `val-workoflow-prod` (SSH alias)
- **User:** `docker` (commands run via `sudo -iu docker`)
- **Remote path:** `/home/docker/docker-setups/workoflow-metrics`
- **Grafana URL:** `http://val-workoflow-prod:3030` (not publicly exposed)
- **Access via SSH tunnel:** `ssh -L 3030:localhost:3030 val-workoflow-prod` → http://localhost:3030

### Commit & Deploy Workflow
```bash
# 1. Make changes to dashboards/*.json or other files
# 2. Commit
git add dashboards/my-dashboard.json
git commit -m "fix: description of change"
# 3. Deploy (pushes to GitHub, pulls on remote, restarts Grafana)
./scripts/deploy.sh deploy
```

The `deploy` command does: `git push` → SSH to prod → `git pull` → `docker compose up -d`.

### Other deploy.sh Commands
- `./scripts/deploy.sh setup` — First-time setup: clones repo on remote, copies `.env`, starts Grafana
- `./scripts/deploy.sh restart` — Restart Grafana container only
- `./scripts/deploy.sh logs` — Tail Grafana logs
- `./scripts/deploy.sh status` — Show container status

### Dashboard Auto-Reload
Dashboards are provisioned from `dashboards/*.json` → mounted at `/var/lib/grafana/dashboards`. Grafana auto-reloads on container restart. No manual import needed.

## Architecture

- **Docker Compose** with a single Grafana container (grafana:11.5.2)
- Grafana joins two external Docker networks to reach databases by container hostname:
  - `n8n_default` — n8n Postgres databases
  - `workoflow-integration-platform_workoflow` — Workoflow MariaDB
- Credentials stored in `.env` on the production server (not in git)

## Datasources

| UID | Type | What It Contains |
|---|---|---|
| `workoflow-mariadb` | MySQL (MariaDB) | audit_log, user, organisation, prompt |
| `litellm-postgres` | PostgreSQL | LiteLLM_DailyUserSpend, LiteLLM_SpendLogs |
| `n8n-internal` | PostgreSQL | execution_entity, workflow_entity, insights_by_period, insights_metadata |
| `n8n-custom` | PostgreSQL | n8n_chat_histories, chat_user_feedback_rating |

See `DATASOURCES.md` for full panel-by-panel documentation.

## Key SQL Gotchas

- **LiteLLM `date` column is text** — always cast `date::timestamp` in `$__timeFilter()` and `SELECT`
- **n8n insights columns** — `"periodStart"` (not periodStartDate), `"metaId"` (not metadataId), `type` is on `insights_by_period` (not metadata)
- **n8n_chat_histories** — column is `session_id` (not "sessionId"), no timestamp column exists
- **chat_user_feedback_rating** — timestamp column is `"timestamp"` (not created_at)
- **audit_log** — `execution_id` is a real column (not inside JSON `data`), `organisation_id` is on audit_log (not on user table)
- **MariaDB JSON** — use `JSON_UNQUOTE(JSON_EXTRACT(data, '$.field'))` syntax
- **Postgres quoted identifiers** — table names like `"LiteLLM_DailyUserSpend"` need double quotes

## Git

- **Repo:** `valantic-CEC-Deutschland-GmbH/workoflow-metrics`
- **Branch:** `main` (deploy directly from main)
