# 07 - Dynamic Workflows (CLI)

The same flow as [`07-dynamic-workflows/`](../07-dynamic-workflows/), with the **`ant`** CLI: the lead agent writes a workflow, the server runs it in the background, and you follow the run's phases and threads on the stream.

## The agent file

```markdown
---
name: Contract Review Lead (workflows, CLI)
model: claude-opus-5-5
tools:
  - type: agent_toolset_20260401
multiagent:
  type: multiagent_20261001
  workflows:
    type: enabled
    predefined_agents:
      - ./contract-reader.md      # resolved to its id + version by ant apply
  subagents:
    type: disabled
---

You review supplier contracts for Brightloom Inc. ... When you're asked to review more than a
few contracts, start a workflow run instead of reading them yourself. ...
```

```bash
ant apply agent.md environment.yaml   # creates contract-reader.md first
```

## Files

- `dynamic_workflows_cli.ipynb` — step-by-step notebook
- `agent.md`, `contract-reader.md`, `environment.yaml` — written by the notebook
- `claude-lock.json`, `outputs/` — written when you run it; gitignored

## Setup

Requires the `ant` CLI (>= 1.40) installed and authenticated — see [`01-basics-cli/`](../01-basics-cli/#setup). Run cells top to bottom; a run takes a few minutes, with a $5 session budget.
