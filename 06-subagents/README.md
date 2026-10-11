# 06 - Subagents

A lead agent hands work to specialist **subagents**, each running in its own **session thread**: isolated history, shared sandbox, shared budget, its own model.

## What you'll learn

- The `multiagent` block (`multiagent_20261001`): subagents, dynamic workflows, advisor
- Predefined subagents (agents you created and list) vs inline subagents (defined on the fly by the lead)
- Follow the **primary thread**: `session.thread_created`, `agent.thread_message_sent` / `_received`, `session.thread_status_*`
- Why only `session.status_idle` (no thread running) ends the turn, not `session.thread_status_idle`
- Inspect threads, drill into one thread's events, read per-thread usage
- Follow up with the same subagent (threads persist), and archive threads (25 child threads max)

## The configuration

```python
lead = client.beta.agents.create(
    name="Contract Review Lead",
    model="claude-opus-5-5",
    system="... Delegate the liability review to the Liability Reviewer and the renewal review to the Renewal Reviewer ...",
    tools=[{"type": "agent_toolset_20260401"}],
    multiagent={
        "type": "multiagent_20261001",
        "subagents": {
            "type": "enabled",
            "inline_agents": {"type": "disabled"},          # only the agents listed below
            "predefined_agents": [
                {"type": "agent", "id": liability_reviewer.id, "version": liability_reviewer.version},
                {"type": "agent", "id": renewal_reviewer.id, "version": renewal_reviewer.version},
            ],
        },
        "workflows": {"type": "disabled"},                  # on by default; module 07
    },
)
```

Subagents can't have a `multiagent` block themselves: delegation is one level deep.

## Files

- `subagents.ipynb` — a lead (Opus) splits the contract review in [`../sample-data/contracts/`](../sample-data/contracts/) between two Haiku specialists, merges their findings into a memo, and follows up with one of them

## Setup

Shared `.env` at the project root. Dependencies are installed in the first cell. Run cells top to bottom.

CLI version: [`06-subagents-cli/`](../06-subagents-cli/).
