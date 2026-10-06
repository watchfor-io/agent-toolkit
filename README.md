# WatchFor Agent Toolkit

<a href="https://ora.ai/scan/watchfor.io"><img src="https://ora.ai/api/badge/watchfor.io" alt="ora agent-readiness score" height="76" /></a> <a href="https://is-agentic.com/scan/watchfor.io"><img src="https://watchfor.io/api/badge/is-agentic" alt="is-agentic agent readiness score" height="76" /></a> <a href="https://webmcp.ora.ai/watchfor.io"><img src="https://watchfor.io/api/badge/webmcp" alt="WebMCP audit score" height="76" /></a>

Agent-facing tooling for **[WatchFor](https://watchfor.io)** — uptime & infrastructure monitoring built to be operated by AI agents as comfortably as by humans.

WatchFor monitors websites, APIs, SSL certificates, DNS, email, cron jobs, MCP servers and scripted browser journeys (Playwright) from multiple regions worldwide, with **multi-location confirmed alerting**: a failure is only declared *Down* once several locations agree, so alerts reflect real outages rather than one bad network path.

## What's in this repo

| File | Purpose |
| --- | --- |
| [`AGENTS.md`](./AGENTS.md) | Instructions for AI agents: when to use WatchFor, how to authenticate, first calls to make |
| [`skills/watchfor-monitoring/SKILL.md`](./skills/watchfor-monitoring/SKILL.md) | Agent skill for monitoring tasks — install with `npx skills add watchfor-io/agent-toolkit` |
| [`plugin.json`](./plugin.json) | [Agent Plugins](https://agent-plugins.org) manifest |
| [`mcp.json`](./mcp.json) | MCP server configuration (product + documentation servers) |

## Agent surfaces

All surfaces share the same auth (org-scoped API key or OAuth 2.1) and per-plan rate limits:

- **REST API** — `https://watchfor.io/api/v1` · [OpenAPI 3.1 spec](https://watchfor.io/openapi.json) (84 operations, cursor pagination, `Idempotency-Key`, typed JSON errors)
- **MCP server** (59 tools, Streamable HTTP) — `https://watchfor.io/api/mcp` · [manifest](https://watchfor.io/.well-known/mcp.json) · [Smithery listing](https://smithery.ai/servers/hello-65wl/watchfor)
- **Documentation MCP server** (read-only, no auth) — `https://watchfor.io/api/docs-mcp`
- **A2A agent** (18 skills, JSON-RPC) — `https://watchfor.io/api/a2a` · [Agent Card](https://watchfor.io/.well-known/agent-card.json)
- **No-auth sandbox** — `https://watchfor.io/api/v1/sandbox` (sample data in exact production shapes)
- **Prometheus metrics** (Pro plan and above) — `https://watchfor.io/api/v1/metrics` · for the user's own Prometheus/Grafana, not for agents to poll · [docs](https://watchfor.io/docs/api/prometheus) · [exporter](https://github.com/watchfor-io/prometheus-exporter)
- **WebMCP (in-page tools)** — nine tools registered on `document.modelContext` across watchfor.io, so an agent-capable browser can search the docs, open the free tools and start a website audit without an account or a key · [WebMCP audit](https://webmcp.ora.ai/watchfor.io)

### Live diagnostics

Agents can also *investigate*, not just read: 24 on-demand checks run from the probe
fleet — DNS, DNS propagation, DNS health, TLS grade, mail server grade (SMTP, IMAP, POP3), HTTP
headers, ping, traceroute, ports,
blocklists, e-mail policy, Core Web Vitals, global CDN analysis and more — plus
`diagnose_target`, which bundles the common ones into a single verdict. Graded checks
return the dashboard's A+ to F grade with what capped it and the fix. A model can
reason about a website; it has no machine in 20 locations. Runs share the plan
allowance with the dashboard's Toolbox.

- REST — `POST /v1/diagnostics/{slug}` · catalog at `GET /v1/diagnostics`
- MCP — `list_diagnostics`, `run_diagnostic`, `diagnose_target`
- A2A — the `diagnose` skill

### Browser checks (Playwright)

An agent can write a plain `@playwright/test` script for a login, search or
checkout, run it once in a sandboxed browser before anything is saved, read
the failed step with its error, and save the passing script as a scheduled
[browser check](https://watchfor.io/playwright-monitoring). Secrets stay in
organization variables (`process.env.NAME`) and are masked in results.
HTTP, API and MCP checks use the same org secrets as `{{NAME}}` in auth
fields, headers and request bodies; credentials stored on a check are
write-only and read back as `"[redacted]"`.

- REST — `POST /v1/playwright/test-runs`, `GET /v1/monitors/{id}/runs`
- MCP — `test_playwright_script`, `get_playwright_run`, `list_playwright_runs`, then `create_monitor` with `type: "playwright"`
- A2A — `manage-monitor` with `action: "test-script"`

## SDKs

Official zero-dependency clients + `watchfor` CLI, mirroring the REST API resource for resource:

| Language | Install | Version |
| --- | --- | --- |
| TypeScript / Node | `npm install watchfor` | [![npm](https://img.shields.io/npm/v/watchfor)](https://www.npmjs.com/package/watchfor) |
| Python | `pip install watchfor` | [![PyPI](https://img.shields.io/pypi/v/watchfor)](https://pypi.org/project/watchfor/) |
| Ruby | `gem install watchfor` | [![Gem](https://img.shields.io/gem/v/watchfor)](https://rubygems.org/gems/watchfor) |

## Quickstart (60 seconds)

```bash
# 1. Try the sandbox — no account, no key:
curl https://watchfor.io/api/v1/sandbox/summary

# 2. Sign up free (no credit card), create an API key in Settings → API keys, then:
curl https://watchfor.io/api/v1/summary -H "Authorization: Bearer wf_live_YOUR_KEY"

# 3. Or connect an MCP client with OAuth — no manual key at all:
claude mcp add --transport http watchfor https://watchfor.io/api/mcp
```

## Links

- Website: <https://watchfor.io>
- API docs: <https://watchfor.io/docs/api>
- Agent quickstart: <https://watchfor.io/agents.md>
- LLM site overview: <https://watchfor.io/llms.txt>
- Machine-readable pricing: <https://watchfor.io/pricing.md>
- Changelog: <https://watchfor.io/changelog>
- Press kit and fact sheet: <https://watchfor.io/press> (Markdown: <https://watchfor.io/press.md>)

## License

[MIT](./LICENSE)
