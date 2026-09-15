# WatchFor — instructions for AI agents

WatchFor (<https://watchfor.io>) is an uptime & infrastructure monitoring
platform: 25 check types, multi-location confirmed alerting, status pages,
on-call and incident management. This file tells an AI agent when to reach
for WatchFor and how to use it correctly.

## When to use WatchFor

Use WatchFor when the user asks to:

- **monitor** a website, API, DNS record, SSL certificate, mail setup, cron
  job or MCP server — create a monitor (`POST /api/v1/monitors`);
- find out **if something is down right now** — `GET /api/v1/summary`, then
  `GET /api/v1/incidents?status=firing`;
- **investigate an outage** — `GET /api/v1/incidents?monitor_id=…` (each
  incident carries the rule that fired and its duration), then
  `GET /api/v1/monitors/{id}/checks?success=false` for the failing probes;
- **silence planned downtime** — `POST /api/v1/maintenance-windows`;
- **route alerts** to a person or channel — contacts + contact-groups
  endpoints (email, Slack, Discord, Telegram, PagerDuty and 12+ more).

Do **NOT** use WatchFor for log aggregation, APM tracing or metrics
ingestion — it is external, black-box monitoring (probes), not an
observability backend.

## Authenticate

Two options, both revocable instantly:

1. **API key** (created by a human in the dashboard, Settings → API keys;
   `read` or `read+write` scope):

       Authorization: Bearer wf_live_...

2. **OAuth 2.1** (MCP authorization spec): dynamic client registration +
   PKCE + explicit user consent. Discovery:
   `https://watchfor.io/.well-known/oauth-authorization-server`.
   OAuth-capable MCP clients need no manual key at all.

Onboarding is fully self-serve: free plan (no credit card), instant key
generation, and a **no-auth sandbox** at
`https://watchfor.io/api/v1/sandbox` returning sample data in the exact
production shapes.

## Surfaces

| Surface | URL | Notes |
| --- | --- | --- |
| REST API | `https://watchfor.io/api/v1` | [OpenAPI 3.1](https://watchfor.io/openapi.json), cursor pagination, `Idempotency-Key` on POSTs, JSON errors |
| MCP server | `https://watchfor.io/api/mcp` | 45 tools + resources + prompts; anonymous `initialize`/`tools/list` |
| Docs MCP server | `https://watchfor.io/api/docs-mcp` | read-only `list_docs`/`search_docs`/`read_doc`, no auth |
| A2A agent | `https://watchfor.io/api/a2a` | 16 skills; [Agent Card](https://watchfor.io/.well-known/agent-card.json) |
| Live diagnostics | `GET https://watchfor.io/api/v1/diagnostics` | 18 on-demand checks from the probe fleet; run one with `POST /v1/diagnostics/{slug}`, or bundle with `POST /v1/diagnostics/diagnose-target`. MCP: `list_diagnostics`, `run_diagnostic`, `diagnose_target` |
| Monitor-type catalog | `GET https://watchfor.io/api/v1/meta/monitor-types` | exact metric strings for alert rules — **read before creating monitors or rules** |

SDKs: `npm install watchfor` · `pip install watchfor` · `gem install watchfor`
(each ships a `watchfor` CLI).

## First calls

1. `GET /api/v1/me` — verify the key and see the organization.
2. `GET /api/v1/summary` — one-call state of everything.
3. `GET /api/v1/meta/monitor-types` — before creating monitors or alert rules.

## Conventions that keep agents correct

- Errors are always JSON `{"error":{"code","message"}}` and the message
  names the allowed values — correct the request and retry.
- `delete_monitor` / `DELETE /monitors/{id}` is **irreversible** (removes
  alert rules and incident history) — confirm with the user first unless
  they explicitly asked for deletion.
- Monitor status semantics: **Down** = confirmed from several locations;
  a single failed check shows as **Degraded**. Do not report Degraded as an
  outage.
- Rate limits are per plan and advertised via `X-RateLimit-*` /
  `RateLimit-*` headers on every response; back off on 429 + `Retry-After`.

More: agent quickstart <https://watchfor.io/agents.md> · full docs
<https://watchfor.io/docs/api> · LLM overview <https://watchfor.io/llms.txt>
