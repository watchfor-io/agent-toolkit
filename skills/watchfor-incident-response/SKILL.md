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
   failures tell different stories). `get_check_details` with a check's
   `run_id` shows everything the probe saw on that run.
4. `list_notifications` — confirm alerts were delivered (channel + status).
   Nothing sent? `get_monitor` → `has_notification_route: false` means the
   incident reached nobody (empty contact group, no channel, or channels
   that take warnings only) — say so and point to the group. `get_on_call`
   says who is on call now and whether they can actually be paged — check
   it before saying someone was notified.
5. `list_activity` with `since` / `until` — who changed what (monitors,
   rules, channels, maintenance), to line the incident up with recent
   config changes.
6. Not sure where it breaks? `diagnose_target` measures a host or URL live
   in one call (DNS, propagation, TLS grade, HTTP headers, ping from up to 3
   regions; needs a plan with location selection). `run_diagnostic` runs one
   live check from the probe fleet (e.g. `dns-propagation`, `tls-grade`,
   `port-checker`), also on targets you don't monitor; `list_diagnostics`
   lists the checks, their options and the remaining allowance. These send
   real traffic: write scope, and each spends plan allowance.
7. What customers see: `list_status_pages` → `get_status_page` (overall
   status, every component and the monitors behind it) and
   `list_announcements` (published incidents and maintenance with their
   update timeline).

Report per incident: monitor, rule, duration, per-location evidence, most
likely cause, one concrete next step.

## Act (write)

- `acknowledge_incident` — marks a human/agent is on it; **stops escalation
  chains**. Acknowledging an already-resolved incident returns 409.
- `resolve_incident` — closes it; use when the underlying cause is fixed.
- `create_maintenance_window` — for planned work: suppresses alerts and
  excludes the window from uptime, **without pausing checks**. Prefer this
  over pausing monitors.
- `set_incident_uptime_exclusion` — once resolved, mark a false positive
  (`excluded: true` with a `reason`): it stays on record but leaves uptime,
  SLA, reports and the public status page. Firing or acknowledged → 409,
  resolve first; `excluded: false` undoes it.

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
