# 03 - Tools and Permissions (CLI)

The same flow as [`03-tools-and-permissions/`](../03-tools-and-permissions/), driven with the **`ant`** CLI. The tool configuration lives in the agent's frontmatter; answering tool calls happens in a Python loop that reads `ant`'s event stream.

## What you'll learn

- Declare toolset overrides, permission policies, and a custom tool's schema in `agent.md`
- Answer pauses with `ant beta:sessions:events send`: `user.custom_tool_result` and `user.tool_confirmation`
- Read `stop_reason` to know whether the turn is over (`requires_action` means keep reading)
- Update an agent by editing its file and running `ant apply` again — the plan shows the diff, and the agent gets a new version

## Tools in the frontmatter

```markdown
---
name: Inventory Agent (CLI)
model: claude-opus-5-5
tools:
  - type: agent_toolset_20260401
    configs:
      - name: web_search
        enabled: false
      - name: bash
        permission_policy:
          type: always_ask
  - type: custom
    name: lookup_stock
    description: Look up how many units of a product are in the warehouse. ...
    input_schema:
      type: object
      properties:
        sku: {type: string}
      required: [sku]
---

You are an inventory assistant. Use the lookup_stock tool for stock levels.
```

## Answering from the shell

```bash
ant beta:sessions:events send --session-id "$SID" \
  --event "{type: user.tool_confirmation, tool_use_id: $EVENT_ID, result: allow}"
```

To approve calls by hand instead, attach an interactive terminal: `ant beta:sessions connect "$SID"`.

## Files

- `tools_and_permissions_cli.ipynb` — step-by-step notebook
- `environment.yaml` / `agent.md` — written by the notebook (`agent.md` ends with `bash` under `auto`, its final state)
- `claude-lock.json` — written by `ant apply`; gitignored in this repo

## Setup

Requires the `ant` CLI (>= 1.40) installed and authenticated — see [`01-basics-cli/`](../01-basics-cli/#setup). Run cells top to bottom.
