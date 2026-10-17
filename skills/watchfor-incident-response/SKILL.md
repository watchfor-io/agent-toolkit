---
name: watchfor-incident-response
description: Respond to live incidents on WatchFor (watchfor.io) — triage what is firing, explain the root cause from probe data, acknowledge and resolve incidents, and silence planned downtime with maintenance windows. Use when the user says something is down, asks "what's broken", wants an incident acknowledged/resolved, or needs planned maintenance to stop paging.
---

# WatchFor incident response

Triage → explain → act. WatchFor incidents are **multi-location confirmed**
(a firing incident means several probes agree), so treat every firing
incident as real.

## Setup

MCP server `https://watchfor.io/api/mcp` (OAuth 2.1 — no manual key — or
`Authorization: Bearer wf_live_…`). REST equivalent: `https://watchfor.io/api/v1`.
Acknowledge/resolve need `write` scope.

## Triage (read)

1. `get_summary` — counts by status + active incidents in one call.
2. `list_incidents` `status=firing` — each incident carries the **rule that
   fired** (metric, operator, threshold), severity and duration.
3. For each affected monitor: `get_monitor_checks` with `success=false` —
   failing probes with per-location error detail (timeouts vs 5xx vs DNS
   failures tell different stories).
4. `list_notifications` — confirm alerts were delivered (channel + status).

Report per incident: monitor, rule, duration, per-location evidence, most
likely cause, one concrete next step.

## Act (write)

- `acknowledge_incident` — marks a human/agent is on it; **stops escalation
  chains**. Acknowledging an already-resolved incident returns 409.
- `resolve_incident` — closes it; use when the underlying cause is fixed.
- `create_maintenance_window` — for planned work: suppresses alerts and
  excludes the window from uptime, **without pausing checks**. Prefer this
  over pausing monitors.

## Post-mortem (write)

- `get_postmortem` — the incident's post-mortem draft (`postmortem: null`
  when none exists yet).
- `update_postmortem` — creates the draft on first call, then updates it:
  `title`, `severity`, `root_cause_category`, `status` (`draft` /
  `completed`), `sections` (only the named ones are replaced) and
  `add_timeline` / `add_action_items` (append, repeats skipped). Write only
  facts from `get_incident` (timeline, root cause, duration); mark unknowns
  as not known yet instead of guessing, and label follow-ups as suggestions.

## Rules

- Never resolve an incident you have not explained — acknowledge instead.
- A single failed check = **Degraded**, not an outage; do not page people
  over it.
- On HTTP 429 back off per `Retry-After`.

Docs: <https://watchfor.io/docs/api> · Incident model: <https://watchfor.io/docs/core-concepts>
