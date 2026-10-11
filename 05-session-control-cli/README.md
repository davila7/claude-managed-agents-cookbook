# 05 - Session Control (CLI)

The same controls as [`05-session-control/`](../05-session-control/), with the **`ant`** CLI.

## Commands

```bash
# Budget at creation (cents as a string) — and change it later
ant beta:sessions create --agent "$AGENT" --environment-id "$ENV" \
  --budget '{"type": "limit", "max_list_cost": {"amount": "200", "currency": "USD"}}'
ant beta:sessions update --session-id "$SID" \
  --budget '{"type": "limit", "max_list_cost": {"amount": "500", "currency": "USD"}}'

# Interrupt and redirect in one request
ant beta:sessions:events send --session-id "$SID" <<'YAML'
events:
  - type: user.interrupt
  - type: user.message
    content:
      - type: text
        text: Stop that. Instead, ...
YAML

# History and audit
ant beta:sessions:events list --session-id "$SID" --type agent.tool_use --format jsonl

# Inspect, follow by hand, close
ant beta:sessions retrieve --session-id "$SID" --transform usage
ant beta:sessions connect "$SID"
ant beta:sessions archive --session-id "$SID"
```

A `system.message` must be the **last** event in its request. After an interrupt, the interrupted turn ends with an `end_turn` idle; keep reading until your redirect has run.

## Files

- `session_control_cli.ipynb` — step-by-step notebook
- `environment.yaml` / `agent.md` — written by the notebook
- `claude-lock.json` — written by `ant apply`; gitignored

## Setup

Requires the `ant` CLI (>= 1.40) installed and authenticated — see [`01-basics-cli/`](../01-basics-cli/#setup). Run cells top to bottom.
