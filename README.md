# Workoflow Metrics Dashboard

Grafana-based metrics dashboard for monitoring the Workoflow platform. Connects directly to production databases — no collector scripts needed.

## Architecture

Grafana joins two existing Docker networks to access databases by container hostname:

- **n8n_default** — N8N Postgres (executions, workflows, chat histories)
- **workoflow-integration-platform_workoflow** — MariaDB (platform data)

## Data Sources

| Name | Type | Host | Database |
|------|------|------|----------|
| N8N Internal Postgres | PostgreSQL | n8n-postgres-1:5432 | (n8n internal) |
| LiteLLM Postgres | PostgreSQL | litellm-postgres:5432 | litellm |
| Workoflow MariaDB | MySQL | mariadb:3306 | workoflow_db |
| N8N Custom Tables | PostgreSQL | n8n-postgres-1:5432 | postgres |

## Dashboards

1. **LLM Costs & Usage** — Daily spend, token consumption, latency, provider distribution
2. **Workflow Executions** — Execution counts, failure rates, top workflows, runtime trends
3. **Tool Usage Analytics** — Tool call volumes, failure rates, success/fail breakdown
4. **User Activity & Platform Health** — Authentications, registrations, chat conversations, feedback ratings

## Setup

```bash
# 1. Clone the repo
git clone git@github.com:valantic/workoflow-metrics.git
cd workoflow-metrics

# 2. Configure credentials
cp .env.example .env
# Edit .env with production values

# 3. Start Grafana
docker compose up -d

# 4. Access at http://localhost:3030
```

## Deployment

```bash
# Initial setup on production
./scripts/deploy.sh setup

# Deploy changes
./scripts/deploy.sh deploy

# Other commands
./scripts/deploy.sh restart
./scripts/deploy.sh logs
./scripts/deploy.sh status
```

## Ports

- **3030** — Grafana UI (mapped from container port 3000)
