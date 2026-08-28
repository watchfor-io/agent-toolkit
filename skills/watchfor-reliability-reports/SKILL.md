---
name: watchfor-reliability-reports
description: Build uptime and reliability reports from WatchFor (watchfor.io) monitoring data — uptime percentages, incident counts, MTTR/MTBF, the least reliable monitors, and plain-language summaries for a period (24h/7d/30d). Use when the user asks "how reliable was X", wants an SLA/uptime report, a weekly ops summary, or to find their flakiest service.
---

# WatchFor reliability reports

Turn raw monitoring history into an answerable report: how reliable were
we, what broke most, and what should we fix first.

## Setup

MCP server `https://watchfor.io/api/mcp` (OAuth 2.1 or
`Authorization: Bearer wf_live_…`). Everything here is read-only — a
`read`-scope key is enough. REST: `https://watchfor.io/api/v1`.

## Build the report

1. `get_summary` — current state (up/down counts, health score) as the
   headline.
2. `get_incident_stats` with `period` (`24h`, `7d`, `30d`) — totals by
   severity, MTTR, MTBF and the most unstable monitors.
3. Per key monitor: `get_monitor_uptime` with the same period — uptime %
   (maintenance windows are already excluded from uptime).
4. `list_incidents` for the period — durations and the rules that fired;
   group by monitor to find repeat offenders.

## Structure the output

- **Headline:** overall uptime, incident count, worst incident (duration).
- **Trends:** MTTR/MTBF vs the previous period if asked.
- **Offenders:** top monitors by incident count or lowest uptime, each with
  its dominant failure mode (from the fired rules).
- **One recommendation:** the highest-leverage fix (e.g. the monitor whose
  incidents account for most downtime).

## Rules

- State the period explicitly in every number you report.
- Uptime excludes maintenance windows by design — say so when relevant.
- Do not extrapolate SLA compliance beyond the requested window.

Docs: <https://watchfor.io/docs/api> · Status model: <https://watchfor.io/docs/core-concepts>
