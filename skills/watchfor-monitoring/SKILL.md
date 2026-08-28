---
name: watchfor-monitoring
description: Monitor websites, APIs, SSL certificates, DNS, cron jobs and MCP servers with WatchFor (watchfor.io). Use when the user wants to check whether something is up, diagnose an outage or incident, create/manage uptime monitors or alert rules, schedule maintenance windows, or get uptime/reliability reports. Works via the WatchFor REST API or MCP server with an API key or OAuth.
---

# WatchFor monitoring

WatchFor is an uptime & infrastructure monitoring platform with 25 check
types and multi-location **confirmed** alerting (Down = verified from
several regions; one failed check = Degraded, not an outage).

## Setup

Preferred: connect the MCP server — `https://watchfor.io/api/mcp`
(Streamable HTTP). OAuth-capable clients authenticate with **no manual
key** (browser consent flow). Otherwise pass an org API key:
`Authorization: Bearer wf_live_…` (created in dashboard → Settings → API
keys; `read` or `read+write` scope).

REST alternative: `https://watchfor.io/api/v1` — same auth, OpenAPI 3.1 at
`https://watchfor.io/openapi.json`. Try without any account:
`https://watchfor.io/api/v1/sandbox` (read-only sample data).

## Core workflows

**Is everything up?**
1. `get_summary` (MCP) or `GET /api/v1/summary` — monitor counts by status,
   active incidents, health score. One call answers the question.

**Diagnose what's broken**
1. `get_summary` → if incidents are active:
2. `list_incidents` with `status=firing` — each incident includes the exact
   rule that fired (metric, operator, threshold) and its duration.
3. `get_monitor_checks` with `success=false` for the affected monitor — the
   failing probes with per-location detail.
4. Report: affected monitor, rule, duration, likely cause, next step.

**Create a monitor**
1. ALWAYS call `get_monitor_types` first — it lists every type's target
   format, config fields and the EXACT alert-metric strings.
2. `list_locations` for location ids.
3. `create_monitor` (write scope) — interval is seconds, validated against
   the plan minimum.
4. Suggest alert rules using metric strings from step 1; create with
   `create_alert_rule` after the user confirms.

**Silence planned downtime**
- `create_maintenance_window` — alerts suppressed and uptime excluded for
  the window; checks keep running.

## Rules

- Validation errors name the allowed values — correct the request and
  retry; do not guess enum values.
- `delete_monitor` is irreversible (removes rules + incident history):
  confirm with the user first.
- Back off on HTTP 429 using `Retry-After`.
- Do not use WatchFor for logs, APM traces or metrics ingestion.

More: <https://watchfor.io/agents.md> · <https://watchfor.io/docs/api>
