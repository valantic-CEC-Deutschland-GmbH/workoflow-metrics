# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## 2026-02-10

### Fixed
- **Time saved queries using unreliable insights tables** — Switched "Per-Workflow Time Saved" time series and "Top Workflows by Time Saved" bar chart from `insights_by_period`/`insights_metadata` to `execution_entity` joined with `workflow_entity.settings->timeSavedPerExecution`. Calculates time saved as successful execution count × configured minutes per execution, which is more accurate and avoids missing data from insights aggregation delays.

## 2026-02-09

### Fixed
- **Feedback rating gauge showing green at wrong position** — Gauge scale was 0-5 but actual ratings are 1-3 (1=Bad, 2=Fine, 3=Good). Fixed scale to 1-3 with correct thresholds. Dismissed ratings (value=0) were included in AVG, dragging it down. Added `AND rating_value > 0` filter. Affects: User Activity "Avg User Feedback Rating" and Executive Summary "User Satisfaction" gauges.
- **Rating distribution bar chart included dismissed ratings** — Bar chart showed a "0" bar for dismissed ratings. Added `AND rating_value > 0` filter and CASE labels (1 - Bad, 2 - Fine, 3 - Good). Affects: User Activity "User Feedback Rating Distribution".
- **LiteLLM time-series panels showing "No data"** — The `date` column in `LiteLLM_DailyUserSpend` is type `text`, not `timestamp`. All LiteLLM queries now cast `date::timestamp` so Grafana can use it as a time axis. Affects: Daily Spend by Model, Token Consumption, Request Volume, Failed Requests, Cost vs Usage Trend, and all stat panels using time filters.
- **n8n insights panels showing "No data"** — Column names were wrong: `periodStartDate` → `periodStart`, `metadataId` → `metaId`, JOIN used `m.id` instead of `m."metaId"`, and WHERE used `m.type` instead of `p.type`. Affects: Estimated Time Saved, Runtime Trend, Estimated Hours Saved.
- **Chat Conversations showing "No data"** — Column was `session_id` not `"sessionId"`, and table has no timestamp column. Removed time filter. Affects: Executive Summary and User Activity dashboards.
- **User feedback panels showing "No data"** — Timestamp column is `"timestamp"` not `created_at`. Affects: User Satisfaction gauge, Avg Feedback Rating, Rating Distribution.
- **Tool Execution Duration showing "No data"** — JOIN used `JSON_EXTRACT` to match execution IDs but `execution_id` is a real column on `audit_log`. Fixed to use `s.execution_id = c.execution_id`.
- **Top Users by Tool Usage showing "No data"** — JOIN used `u.organisation_id` but `organisation_id` lives on `audit_log`, not `user`. Fixed to use `al.organisation_id`.
- **deploy.sh remote execution** — Fixed SSH command execution and switched to HTTPS clone URL

### Added
- **Executive Summary dashboard** — New CEO-level KPI overview with 12 panels across all datasources: active users, LLM spend, workflows automated, satisfaction, tool executions, cost per request, chat conversations, success rate, activity trends, cost vs usage, top integrations, estimated hours saved
- **Dashboard titles now indicate datasource origin** — Executive Summary (All Sources), LLM Costs & Usage (LiteLLM), Workflow Executions (n8n), Tool Usage Analytics (Integration Platform), User Activity & Platform Health (Integration Platform)
- **DATASOURCES.md** — Full documentation of all 4 datasources, their tables, and every panel across all 5 dashboards with SQL query summaries and known empty panels
- **CLAUDE.md** — Project documentation with deployment workflow, server info, architecture, and SQL gotchas

### Changed
- **README.md** — Added SSH tunnel instructions for accessing Grafana, fixed git clone URL to correct org, updated dashboard list to include Executive Summary

## 2026-01-30

### Added
- **Initial Grafana metrics dashboard** — Docker Compose setup with Grafana 11.5.2 connecting to 4 datasources (workoflow-mariadb, litellm-postgres, n8n-internal, n8n-custom) via external Docker networks
- **LLM Costs & Usage dashboard** — 11 panels: total spend, cost per request, cache hit rate, daily spend by model, token consumption, request volume, latency percentiles, provider distribution, failed requests, spend by API key, week-over-week cost change
- **Workflow Executions dashboard** — 10 panels: total executions, failure rate, avg execution time, daily executions by status, top workflows, top failing workflows, execution mode breakdown, time saved, runtime trend, executions per day
- **Tool Usage Analytics dashboard** — 8 panels: total tool calls, failed calls, executions over time, top tools, success/fail breakdown, failure rate table, execution duration, top users
- **User Activity & Platform Health dashboard** — 12 panels: total users, active orgs, chat conversations, feedback rating, daily authentications, registrations, API requests, prompt library growth, rating distribution, DAU, WAU, new vs returning users
- **Provisioning configuration** — Auto-provisioned datasources and dashboard directory
- **deploy.sh** — Deployment script with setup, deploy, restart, logs, and status commands for `val-workoflow-prod`
