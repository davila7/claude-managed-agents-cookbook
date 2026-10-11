# 03 - Tools and Permissions

Your client takes part in the agent's turn: run custom tools in your own system, and approve or deny built-in tool calls before they execute.

## What you'll learn

- Tune the built-in toolset with `configs`: disable tools, set a `permission_policy` per tool
- Define a **custom tool** — the agent calls it, your code runs it
- Answer pauses: `user.custom_tool_result` for custom tools, `user.tool_confirmation` for `always_ask`
- Read `stop_reason` on `session.status_idle` to know whether the turn is really over
- Let the server judge each call with the `auto` policy, and switch to it by updating the agent (new version)

## Two kinds of pause

| Pause | The agent emits | You answer with |
|---|---|---|
| Custom tool | `agent.custom_tool_use` | `user.custom_tool_result` (`custom_tool_use_id`) |
| Built-in tool under `always_ask` | `agent.tool_use` with `evaluated_permission: "ask"` | `user.tool_confirmation` (`tool_use_id`, `allow` / `deny`) |

## The pattern: answer on arrival

```python
for event in stream:
    match event.type:
        case "agent.custom_tool_use":
            result, is_error = run_my_tool(event.name, event.input)
            send({"type": "user.custom_tool_result", "custom_tool_use_id": event.id,
                  "content": [{"type": "text", "text": result}], "is_error": is_error})
        case "agent.tool_use" if event.evaluated_permission == "ask":
            send({"type": "user.tool_confirmation", "tool_use_id": event.id, "result": "allow"})
        case "session.status_idle":
            if event.stop_reason.type == "requires_action":
                continue        # already answering — keep reading
            break               # end_turn, budget_reached, ...
```

Answering each call as its event arrives (instead of waiting for the `requires_action` idle) keeps working once a session runs several agents, where that idle may never come.

## Permission policies

| Policy | Behavior |
|---|---|
| `always_allow` | Runs with no confirmation (default for the built-in toolset) |
| `always_ask` | Pauses until your client sends `allow` or `deny` |
| `auto` | The server runs it, denies it, or pauses for you |

Custom tools aren't governed by permission policies — your code decides whether to run them.

## Files

- `tools_and_permissions.ipynb` — inventory agent with a custom `lookup_stock` tool and `bash` under `always_ask`, then `auto`

## Setup

Shared `.env` at the project root (same as module 01). Dependencies are installed in the first cell. Run cells top to bottom.

CLI version: [`03-tools-and-permissions-cli/`](../03-tools-and-permissions-cli/).
