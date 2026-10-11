# 05 - Session Control

The controls you need around a long-running session: cap its spend, add context mid-session, interrupt and redirect, reconnect without losing events, audit its history, and resume it after it pauses at its budget.

## What you'll learn

| Control | How |
|---|---|
| Cap the spend | `sessions.create(budget={"type": "limit", "max_list_cost": {"amount": "200", "currency": "USD"}})` — cents as a string, set at creation only |
| Watch the spend | `session.usage` events (right before every idle) and `sessions.retrieve().usage` |
| Add context mid-session | a `system.message` event, sent **last** in the request |
| Change direction | `user.interrupt` + a new `user.message` in one request |
| Reconnect | open the stream → `events.list()` to catch up (remember ids) → tail, skipping seen ids |
| Audit | `events.list(session_id, types=["agent.tool_use"])` |
| Resume at the cap | `stop_reason: budget_reached` → `sessions.update(budget=...)`; the paused work resumes by itself |
| Inspect / close | `sessions.retrieve()`, `sessions.list()`, `sessions.archive()`, `sessions.delete()` |

## Two traps this module defuses

- **The interrupted turn ends with `end_turn`.** After an interrupt, the first `session.status_idle` looks like a normal finish. Only count an `end_turn` that comes after your redirect message is processed.
- **A new stream has no memory.** It only delivers events emitted after it opens. To reconnect, open the stream first, catch up from the history, then tail and de-duplicate by event `id`. Only count an `end_turn` idle that comes after your own message as the turn's end.

## Files

- `session_control.ipynb` — one session that goes through every control above

## Setup

Shared `.env` at the project root. Dependencies are installed in the first cell. Run cells top to bottom.

CLI version: [`05-session-control-cli/`](../05-session-control-cli/).
