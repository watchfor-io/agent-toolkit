---
name: watchfor-monitoring
description: Monitor websites, APIs, SSL certificates, DNS, cron jobs, MCP servers, Playwright browser journeys and Linux servers (hosts running watchfor-agent) with WatchFor (watchfor.io). Use when the user wants to check whether something is up, diagnose an outage or incident, create/manage uptime monitors or alert rules, schedule maintenance windows, or get uptime/reliability reports. Works via the WatchFor REST API or MCP server with an API key or OAuth.
---

# WatchFor monitoring

WatchFor is an uptime & infrastructure monitoring platform with 30 check
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
   the plan minimum. Credentials (mailbox/FTP passwords, HTTP auth
   passwords, bearer tokens, `Authorization` / `Cookie` / `X-API-Key`
   header values) are write-only: reads return `"[redacted]"`; omit them
   (or send the mask back) to keep the stored value, and never save the
   mask as a value. To share a token across checks, ask the user to add it
   as an org secret (Settings → Variables) and reference it as `{{NAME}}`
   in the auth field, header value or request body. Moving a check to
   another host needs the credential again.
4. Suggest alert rules using metric strings from step 1; create with
   `create_alert_rule` after the user confirms.

**Accept a deploy (page integrity monitors)**
- Page integrity monitors compare every check with an accepted baseline
  (scripts, forms, security headers, redirects, visible text). After an
  intended release, `accept_integrity_baseline` (REST
  `POST /v1/monitors/{id}/baseline`, write scope) makes what the newest
  check saw the new baseline and starts a check, so the open findings close
  on it. Accept only after a check has seen the release: `run_check_now`,
  wait until `get_monitor_checks` shows a newer check, then accept —
  otherwise the pre-release state becomes the baseline.
  `get_integrity_baseline` shows when each page was last accepted. Never accept unreviewed security findings (a new
  script source, changed third-party code, a new form target) without the
  user's confirmation — accepting makes an injected script "normal".

**Test a Playwright script before saving it (browser checks)**
1. Write a normal `@playwright/test` file: `test.step(...)` per user-visible
   step, web-first `expect`s, `process.env.TARGET_URL` for the site and
   `process.env.NAME` for organization variables and secrets (never inline
   a password).
2. `test_playwright_script` (write scope; REST `POST /v1/playwright/test-runs`)
   with `script` and `target` — runs it once in a sandboxed browser without
   saving. A script error comes back as `400` naming the line. If the answer
   lists `missing_variables`, ask the user to add them (Settings → Variables,
   secrets for passwords) — do not rewrite the script around them.
3. If it is still running, poll `get_playwright_run` with `test_run_id`. On a
   failure, read the failed step, its error and code frame, then fix and
   test again.
4. Once it passes, `create_monitor` with `type: "playwright"`, the site as
   `target` and `config.script`. Later failures: `list_playwright_runs`
   (`status=failed`) → `get_playwright_run` with `monitor_id` + `run_id`.

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

## Servers (hosts)

A host is a Linux server running the open-source watchfor-agent; the token
is issued in the dashboard (Hosts → New host), everything after that is
readable by agents:

- `list_hosts` (REST `GET /v1/hosts?status=offline`) — who is online, stale
  (no batch for three push intervals, at least 90 s) or offline (10 min),
  with current CPU / memory / disk.
- `get_host` — OS, agent version, hardware, addresses and the host's
  `monitor_id`: use it wherever a monitor id is expected (incidents,
  status-page components, maintenance windows, uptime).
- `get_host_metrics` — time series over 1h–30d: `cpu.usage_pct`,
  `mem.used_pct`, `disk.used_pct` (per mount), `load.1`, `net.rx_bytes_per_s`…
  Answer "was CPU high last night" from data, not guesses.
- A2A: the `check-hosts` skill summarises the fleet in one call.
