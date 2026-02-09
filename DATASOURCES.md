# Datasources & Panel Reference

## Datasources

| Name | Type | Database | Description |
|---|---|---|---|
| `workoflow-mariadb` | MySQL (MariaDB) | workoflow | Main application database: users, organisations, audit logs, prompts |
| `litellm-postgres` | PostgreSQL | litellm | LLM proxy spend tracking: daily spend per user/model, individual request logs |
| `n8n-internal` | PostgreSQL | n8n | n8n workflow engine internals: executions, workflows, insights |
| `n8n-custom` | PostgreSQL | n8n (custom schema) | Chat histories and user feedback from n8n chat widgets |

## Tables

### workoflow-mariadb
- **audit_log** — Every user action (tool calls, logins, API requests). Key columns: `user_id`, `action`, `data` (JSON), `created_at`, `execution_id`, `organisation_id`
- **user** — Registered users. Key columns: `id`, `email`, `created_at`
- **organisation** — Organisations/tenants. Key columns: `id`, `name`, `created_at`
- **prompt** — Prompt library entries. Key columns: `id`, `created_at`

### litellm-postgres
- **"LiteLLM_DailyUserSpend"** — Daily aggregated spend per user/model. Key columns: `date` (text, needs `::timestamp` cast), `api_key`, `model`, `custom_llm_provider`, `spend`, `prompt_tokens`, `completion_tokens`, `api_requests`, `failed_requests`
- **"LiteLLM_SpendLogs"** — Individual request logs. Key columns: `"startTime"`, `"endTime"`, `cache_hit`, `spend`

### n8n-internal
- **execution_entity** — Every workflow execution. Key columns: `"startedAt"`, `"stoppedAt"`, `"workflowId"`, `status`, `mode`
- **workflow_entity** — Workflow definitions. Key columns: `id`, `name`
- **insights_by_period** — Aggregated insights (time saved, runtime). Key columns: `"periodStart"`, `"metaId"`, `type` (0=time saved, 1=runtime), `value`
- **insights_metadata** — Metadata for insights. Key columns: `"metaId"`

### n8n-custom
- **n8n_chat_histories** — Chat conversation messages. Key columns: `session_id` (no timestamp column available)
- **chat_user_feedback_rating** — User feedback ratings. Key columns: `rating_value`, `"timestamp"`

---

## Dashboard Panels

### 1. Executive Summary (All Sources)
**File:** `dashboards/executive-summary.json`

| Panel | ID | Type | Datasource | What It Shows |
|---|---|---|---|---|
| Active Users (30d) | 1 | stat | workoflow-mariadb | Count of distinct users with audit_log activity in last 30 days |
| Total LLM Spend | 2 | stat | litellm-postgres | Sum of all LLM spend in selected period |
| Workflows Automated | 3 | stat | n8n-internal | Count of distinct workflows that had executions in period |
| User Satisfaction | 4 | gauge | n8n-custom | Average feedback rating (0-5 scale). Note: feedback collection was disabled; last data from Nov 2025 |
| Tool Executions | 5 | stat | workoflow-mariadb | Count of `tool_execution.started` audit events |
| Cost per Request | 6 | stat | litellm-postgres | Average LLM spend divided by total API requests |
| Chat Conversations | 7 | stat | n8n-custom | Count of distinct chat sessions (all-time, no time filter — table has no timestamp) |
| Automation Success Rate | 8 | gauge | n8n-internal | Percentage of executions with status='success' |
| Weekly Platform Activity | 9 | timeseries | workoflow-mariadb | Daily active users and total actions from audit_log |
| Cost vs Usage Trend | 10 | timeseries | litellm-postgres | Daily LLM spend overlaid with request count |
| Top 10 Integrations by Usage | 11 | barchart | workoflow-mariadb | Most used audit_log actions (excluding user.* actions) |
| Estimated Hours Saved by Automation | 12 | stat | n8n-internal | Sum of insights_by_period time-saved values converted to hours |

### 2. LLM Costs & Usage (LiteLLM)
**File:** `dashboards/llm-costs-usage.json`

| Panel | ID | Type | Datasource | What It Shows |
|---|---|---|---|---|
| Total Spend (period) | 1 | stat | litellm-postgres | Sum of spend for selected time range |
| Cost per Request (avg) | 2 | stat | litellm-postgres | Average cost per API request |
| Cache Hit Rate | 3 | stat | litellm-postgres | Percentage of requests with cache_hit=true from SpendLogs. Expected: 0% (caching not configured) |
| Daily Spend by Model | 4 | timeseries | litellm-postgres | Stacked bar chart of daily spend broken down by model |
| Token Consumption | 5 | timeseries | litellm-postgres | Daily prompt vs completion token counts |
| Request Volume | 6 | timeseries | litellm-postgres | Daily API request count |
| Latency Percentiles | 7 | timeseries | litellm-postgres | p50 and p95 request latency from SpendLogs |
| Provider Distribution | 8 | piechart | litellm-postgres | Spend breakdown by LLM provider |
| Failed Requests | 9 | timeseries | litellm-postgres | Daily count of failed API requests |
| Spend by User / API Key | 10 | table | litellm-postgres | Table of spend, requests, tokens per API key |
| Week-over-Week Cost Change | 11 | stat | litellm-postgres | Percentage change in spend vs previous week |

### 3. Workflow Executions (n8n)
**File:** `dashboards/workflow-executions.json`

| Panel | ID | Type | Datasource | What It Shows |
|---|---|---|---|---|
| Total Executions (period) | 1 | stat | n8n-internal | Count of all executions in period |
| Failure Rate | 2 | gauge | n8n-internal | Percentage of executions with status='error' |
| Avg Execution Time | 3 | stat | n8n-internal | Average duration (stoppedAt - startedAt) |
| Daily Executions | 4 | timeseries | n8n-internal | Stacked bar chart of daily executions by status (success/error) |
| Top Workflows by Volume | 5 | barchart | n8n-internal | Top 15 workflows ranked by execution count |
| Top Failing Workflows | 6 | table | n8n-internal | Workflows with highest failure counts and rates |
| Execution Mode Breakdown | 7 | piechart | n8n-internal | Distribution of execution modes (webhook, trigger, manual, etc.) |
| Estimated Time Saved by Automation | 8 | timeseries | n8n-internal | Time series of insights_by_period type=0 values (time saved) |
| Cumulative Workflow Runtime Trend | 9 | timeseries | n8n-internal | Time series of insights_by_period type=1 values (runtime) |
| Executions + Distinct Workflows per Day | 10 | timeseries | n8n-internal | Daily execution count with distinct workflow count on secondary axis |

### 4. Tool Usage Analytics (Integration Platform)
**File:** `dashboards/tool-usage.json`

| Panel | ID | Type | Datasource | What It Shows |
|---|---|---|---|---|
| Total Tool Calls | 1 | stat | workoflow-mariadb | Count of tool_execution.started events |
| Failed Tool Calls | 2 | stat | workoflow-mariadb | Count of tool_execution.failed events |
| Tool Executions Over Time | 3 | timeseries | workoflow-mariadb | Daily tool execution count |
| Top Tools by Usage | 4 | barchart | workoflow-mariadb | Top 15 tools by call count (extracted from audit_log JSON data) |
| Tool Success/Fail Breakdown | 5 | piechart | workoflow-mariadb | Distribution of tool_execution.started vs .failed vs .completed |
| Tool Failure Rate | 6 | table | workoflow-mariadb | Per-tool failure rate with gauge visualization |
| Tool Execution Duration (avg/max) | 7 | timeseries | workoflow-mariadb | Average and max tool duration by joining started/completed events via execution_id |
| Top Users by Tool Usage | 8 | table | workoflow-mariadb | Top 20 users by tool call count with organisation name |

### 5. User Activity & Platform Health (Integration Platform)
**File:** `dashboards/user-activity.json`

| Panel | ID | Type | Datasource | What It Shows |
|---|---|---|---|---|
| Total Users | 1 | stat | workoflow-mariadb | Total registered user count |
| Active Organizations | 2 | stat | workoflow-mariadb | Total organisation count |
| Chat Conversations | 3 | stat | n8n-custom | Distinct chat session count (all-time, no time filter) |
| Avg User Feedback Rating | 4 | gauge | n8n-custom | Average rating from feedback table. Note: feedback collection disabled since Nov 2025 |
| Daily Authentications | 5 | timeseries | workoflow-mariadb | Daily magic link authentication count |
| User Registrations | 6 | timeseries | workoflow-mariadb | Daily new user registrations |
| API Requests | 7 | timeseries | workoflow-mariadb | Daily API request count from audit_log |
| Prompt Library Growth | 8 | timeseries | workoflow-mariadb | Daily new prompts created |
| User Feedback Rating Distribution | 9 | barchart | n8n-custom | Bar chart of rating values and their counts |
| Daily Active Users (DAU) | 10 | timeseries | workoflow-mariadb | Daily count of distinct active users |
| Weekly Active Users (WAU) | 11 | timeseries | workoflow-mariadb | Weekly count of distinct active users |
| New vs Returning Users | 12 | timeseries | workoflow-mariadb | Stacked bar of new vs returning users per day |

---

## Known Empty Panels

| Panel | Dashboard | Reason |
|---|---|---|
| User Satisfaction / Avg Feedback Rating | Executive Summary, User Activity | Feedback collection was disabled; last data from November 2025. Will show "No data" for recent time ranges. |
| Cache Hit Rate | LLM Costs & Usage | LiteLLM caching is not configured. Expected value: 0% |
