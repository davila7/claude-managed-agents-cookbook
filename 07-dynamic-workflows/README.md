# 07 - Dynamic Workflows

The agent **writes a workflow** (a program that runs many agents in phases and combines their results), and the server executes it in the background as a **workflow run**. You follow the run on the session's stream.

## What you'll learn

- Turn dynamic workflows on with `multiagent.workflows`, and give the workflow predefined agents
- Guide the agent through its system prompt: when to start a run, how to split the work, what to do when an agent fails, a time limit
- Follow a run: `workflow_run.created`, `phase_started` / `phase_ended`, `status_ended`, and the run's threads (`session.thread_created` with `workflow_run_id`)
- Know when the work is done: **no open runs** and then `session.status_idle` with `end_turn`
- Inspect the run's threads (predefined vs inline agents, per-thread cost), and ask the agent for the workflow it wrote
- Budgets and limits for runs

## The configuration

```python
lead = client.beta.agents.create(
    name="Contract Review Lead (workflows)",
    model="claude-opus-5-5",
    system="... When you're asked to review more than a few contracts, start a workflow run ...",
    tools=[{"type": "agent_toolset_20260401"}],
    multiagent={
        "type": "multiagent_20261001",
        "workflows": {
            "type": "enabled",
            "predefined_agents": [{"type": "agent", "id": reader.id, "version": reader.version}],
        },
        "subagents": {"type": "disabled"},
    },
)
```

No API call starts a run: you describe the work in a `user.message`, and the agent decides.

## Knowing when the work is done

```python
open_runs = {}
for event in stream:
    match event.type:
        case "workflow_run.created":      open_runs[event.workflow_run_id] = event.name
        case "workflow_run.status_ended": open_runs.pop(event.workflow_run_id, None)
        case "session.status_idle":
            if event.stop_reason.type == "requires_action":
                continue                  # answer tool calls on arrival (module 03)
            if event.stop_reason.type == "end_turn" and open_runs:
                continue                  # the lead ended its turn; the run works in the background
            break                         # done (end_turn) — or budget_reached / retries_exhausted / refusal
```

## Limits

| Limit | Value |
|---|---|
| Threads working at once in one run | 64 |
| Agents a workflow starts over a run's life | 1,000 (then `thread_limit_error`) |
| Run lifetime | 24 hours by default, or what the agent sets |
| Runs open at once in a session | 10 |

## Files

- `dynamic_workflows.ipynb` — the contract review from modules 04 and 06, run as a workflow: one Haiku reader per contract, then inline agents check every extraction
- `outputs/` — written by the notebook when it downloads the deliverables (gitignored)

## Setup

Shared `.env` at the project root. Dependencies are installed in the first cell. Run cells top to bottom. A run takes a few minutes; the session budget is $5.

CLI version: [`07-dynamic-workflows-cli/`](../07-dynamic-workflows-cli/).
