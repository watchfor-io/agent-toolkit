# WatchFor Agent Toolkit

[![smithery badge](https://smithery.ai/badge/hello-65wl/watchfor)](https://smithery.ai/servers/hello-65wl/watchfor) <a href="https://ora.ai/scan/watchfor.io"><img src="https://ora.ai/api/badge/watchfor.io" alt="ora agent-readiness score" height="76" /></a> <a href="https://is-agentic.com/scan/watchfor.io"><img src="https://watchfor.io/api/badge/is-agentic" alt="is-agentic agent readiness score" height="76" /></a>

Agent-facing tooling for **[WatchFor](https://watchfor.io)** — uptime & infrastructure monitoring built to be operated by AI agents as comfortably as by humans.

WatchFor monitors websites, APIs, SSL certificates, DNS, email, cron jobs and MCP servers from multiple regions worldwide, with **multi-location confirmed alerting**: a failure is only declared *Down* once several locations agree, so alerts reflect real outages rather than one bad network path.

## What's in this repo

| File | Purpose |
| --- | --- |
| [`AGENTS.md`](./AGENTS.md) | Instructions for AI agents: when to use WatchFor, how to authenticate, first calls to make |
| [`skills/watchfor-monitoring/SKILL.md`](./skills/watchfor-monitoring/SKILL.md) | Agent skill for monitoring tasks — install with `npx skills add watchfor-io/agent-toolkit` |
| [`plugin.json`](./plugin.json) | [Agent Plugins](https://agent-plugins.org) manifest |
| [`mcp.json`](./mcp.json) | MCP server configuration (product + documentation servers) |

## Agent surfaces

All surfaces share the same auth (org-scoped API key or OAuth 2.1) and per-plan rate limits:

- **REST API** — `https://watchfor.io/api/v1` · [OpenAPI 3.1 spec](https://watchfor.io/openapi.json) (57 operations, cursor pagination, `Idempotency-Key`, typed JSON errors)
- **MCP server** (37 tools, Streamable HTTP) — `https://watchfor.io/api/mcp` · [manifest](https://watchfor.io/.well-known/mcp.json) · [Smithery listing](https://smithery.ai/servers/hello-65wl/watchfor)
- **Documentation MCP server** (read-only, no auth) — `https://watchfor.io/api/docs-mcp`
- **A2A agent** (13 skills, JSON-RPC) — `https://watchfor.io/api/a2a` · [Agent Card](https://watchfor.io/.well-known/agent-card.json)
- **No-auth sandbox** — `https://watchfor.io/api/v1/sandbox` (sample data in exact production shapes)

## SDKs

Official zero-dependency clients + `watchfor` CLI, mirroring the REST API resource for resource:

| Language | Install | Registry |
| --- | --- | --- |
| TypeScript / Node | `npm install watchfor` | [npm](https://www.npmjs.com/package/watchfor) |
| Python | `pip install watchfor` | [PyPI](https://pypi.org/project/watchfor/) |
| Ruby | `gem install watchfor` | [RubyGems](https://rubygems.org/gems/watchfor) |

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

## License

[MIT](./LICENSE)
