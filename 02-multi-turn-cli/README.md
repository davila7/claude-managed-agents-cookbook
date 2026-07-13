# 02 - Multi-Turn Conversations (CLI)

The same multi-turn flow as [`02-multi-turn/`](../02-multi-turn/), driven from the terminal with Anthropic's **`ant`** CLI instead of the Python SDK. One section, two flavors: the SDK for application code, the CLI for ad-hoc control-plane work and version-controlled YAML.

## What you'll learn

- Create **one session** and reuse it across turns — the crux of multi-turn
- Conversation history is kept **server-side**; you never resend previous turns
- Wrap the per-turn stream-first choreography in a reusable `chat()` helper

## The pattern

Agents and environments are static — define them as YAML and apply once with `!ant ... create`. The session is created **once** and reused; each turn repeats the stream-first dance. Streaming needs to read events live, so we drive `ant` from Python with `subprocess`:

```python
# One session, created once and kept in session_id
_session = !ant beta:sessions create --agent '{agent_ref}' --environment-id {env_id} --transform id -r
session_id = _session[0]

def chat(message):
    # Open the event stream FIRST (stream-before-send), then send, then read until idle.
    stream = subprocess.Popen(["ant", "beta:sessions:events", "stream",
        "--session-id", session_id, "--format", "yaml"], stdout=subprocess.PIPE, text=True, bufsize=1)
    subprocess.run(["ant", "beta:sessions:events", "send", "--session-id", session_id],
        input=json.dumps({"events": [{"type": "user.message",
            "content": [{"type": "text", "text": message}]}]}), text=True, check=True)
    # ...read agent.message text until session.status_idle...

chat("What's the largest planet in our solar system?")
chat("How many moons does it have?")     # context preserved automatically
```

> **Why Python, not `%%bash`?** ant's `{name}<` process-substitution needs bash ≥ 4.1, but macOS ships bash 3.2; and `--format jsonl` buffers until the stream ends. Reading `--format yaml` from Python `subprocess` is portable and parses each event robustly.

## Session status reference

| Status | Meaning |
|---|---|
| `session.status_idle` | Agent finished its turn — send another message |
| `session.status_terminated` | Session closed — create a new one |

## Files

- `multi_turn_cli.ipynb` — step-by-step notebook running `ant` commands cell by cell
- `env.yaml` / `agent.yaml` — written by the notebook; the version-controlled resource definitions

## Setup

Requires the `ant` CLI installed and authenticated — see [`01-basics-cli/`](../01-basics-cli/#setup) for install and auth details. The setup cell loads `ANTHROPIC_API_KEY` from the shared root `.env` and clears Jupyter's `FORCE_COLOR` (which would otherwise make `ant` emit ANSI color codes into its JSON/YAML and break parsing):

```bash
cp ../.env.example ../.env
# edit ../.env and add your ANTHROPIC_API_KEY
```

Run cells top to bottom.
