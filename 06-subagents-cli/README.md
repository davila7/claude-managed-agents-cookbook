# 06 - Subagents (CLI)

The same flow as [`06-subagents/`](../06-subagents/), with the **`ant`** CLI.

## Resources refer to each other by path

The lead lists its subagents by **file path**. `ant apply agent.md` follows those references, creates the specialists first, fills in their IDs, and pins their versions:

```markdown
---
name: Contract Review Lead (CLI)
model: claude-opus-5-5
tools:
  - type: agent_toolset_20260401
multiagent:
  type: multiagent_20261001
  subagents:
    type: enabled
    inline_agents:
      type: disabled
    predefined_agents:
      - ./liability-reviewer.md
      - ./renewal-reviewer.md
  workflows:
    type: disabled
---

You lead contract reviews for Brightloom Inc. ...
```

## Thread commands

```bash
ant beta:sessions:threads list --session-id "$SID"
ant beta:sessions:threads:events list --session-id "$SID" --thread-id "$THREAD_ID"
ant beta:sessions:threads:events stream --session-id "$SID" --thread-id "$THREAD_ID"
ant beta:sessions:threads archive --session-id "$SID" --thread-id "$THREAD_ID"
```

## Files

- `subagents_cli.ipynb` — step-by-step notebook
- `agent.md`, `liability-reviewer.md`, `renewal-reviewer.md`, `environment.yaml` — written by the notebook
- `claude-lock.json` — written by `ant apply`; gitignored

## Setup

Requires the `ant` CLI (>= 1.40) installed and authenticated — see [`01-basics-cli/`](../01-basics-cli/#setup). Run cells top to bottom.
