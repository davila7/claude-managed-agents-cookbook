# 02 - Multi-Turn Conversations (CLI)

The same multi-turn flow as [`02-multi-turn/`](../02-multi-turn/), driven from the terminal with Anthropic's **`ant`** CLI instead of the Python SDK. One section, two flavors: the SDK for application code, the CLI for ad-hoc control-plane work and version-controlled YAML.

## What you'll learn

- Create **one session** and reuse it across turns — the crux of multi-turn
- Conversation history is kept **server-side**; you never resend previous turns
- Wrap the per-turn stream-first choreography in a reusable `chat.sh` helper
- Reuse the same `$SID` across independent `%%bash` cells

## The pattern

Agents and environments are static — define them as YAML and apply once. The session is created **once** and reused; each turn repeats the stream-first dance:

```bash
# One session, exported as $SID
SID=$(ant beta:sessions create --agent "$AGENT_REF" --environment-id "$ENV_ID" --transform id -r)

# Each turn: open the stream FIRST, then send, then read until idle.
chat() {
  exec {stream}< <(ant beta:sessions:events stream --session-id "$SID" --format yaml)
  ant beta:sessions:events send --session-id "$SID" <<YAML
events:
  - type: user.message
    content: [{type: text, text: $1}]
YAML
  while IFS= read -r -u "$stream" line; do
    [ "$line" = "type: session.status_idle" ] && break   # ready for next message
  done
  exec {stream}<&-
}

chat "What's the largest planet in our solar system?"
chat "How many moons does it have?"     # context preserved automatically
```

## Session status reference

| Status | Meaning |
|---|---|
| `session.status_idle` | Agent finished its turn — send another message |
| `session.status_terminated` | Session closed — create a new one |

## Files

- `multi_turn_cli.ipynb` — step-by-step notebook running `ant` commands cell by cell
- `env.yaml` / `agent.yaml` — written by the notebook; the version-controlled resource definitions
- `chat.sh` — written by the notebook; the reusable per-turn helper

## Setup

Requires the `ant` CLI installed and authenticated — see [`01-basics-cli/`](../01-basics-cli/#setup) for install and auth details. The CLI reads `ANTHROPIC_API_KEY` from the environment, the same key the Python notebooks load from the shared root `.env`:

```bash
cp ../.env.example ../.env
# edit ../.env and add your ANTHROPIC_API_KEY, then export it for the CLI:
export ANTHROPIC_API_KEY=sk-ant-...
```

Run cells top to bottom.
