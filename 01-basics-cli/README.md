# 01 - Basics (CLI)

The same minimal end-to-end example as [`01-basics/`](../01-basics/), driven from the terminal with Anthropic's **`ant`** CLI instead of the Python SDK. One section, two flavors: pick the SDK for application code, the CLI for ad-hoc control-plane work and version-controlled YAML.

## What you'll build

A complete agent flow from scratch:

```
ant apply environment.yaml        →  environment id recorded in claude-lock.json
ant apply agent.md                →  agent id + version recorded in claude-lock.json
ant beta:sessions create --agent  →  new session per run
ant beta:sessions:events stream   →  open stream FIRST
ant beta:sessions:events send     →  send message, triggers agent loop
read loop                         →  handle responses until idle
```

## CLI for the control plane, SDK for the data plane

| | Control plane → `ant` | Data plane → SDK / CLI |
|---|---|---|
| Resources | agents, environments | sessions, events |
| Cadence | Once per deploy / ad-hoc | Every task / every turn |
| Lives in | `agent.md` / `environment.yaml` in your repo + CI | Application code |

Agents and environments are relatively static — declare them in files, check them into your repo, and reconcile with `ant apply`. Sessions are dynamic — created per run and streamed.

## `ant apply` and the lockfile

`ant apply` compares your files with the API, shows a plan, and creates or updates resources to match. It records each resource's id (and the agent's version) in `claude-lock.json`, so running it again is a no-op — *create once, reuse the ID* enforced by the tool:

```bash
ant apply environment.yaml agent.md --yes --lock-file claude-lock.json
```

The agent file is Markdown with YAML frontmatter; the body is the system prompt:

```markdown
---
name: Hello World Agent (CLI)
model: claude-opus-5-5
tools:
  - type: agent_toolset_20260401
---

You are a helpful assistant. Keep your answers concise.
```

> In your own project, **commit `claude-lock.json`** so the next apply updates the same resources. This cookbook gitignores it because it holds your organization and workspace ids — every reader gets their own.

## Core concepts

| Object | Created | Purpose |
|--------|---------|---------|
| **Environment** | Once | Sandboxed container where tools run (bash, file ops, code execution) |
| **Agent** | Once | Persisted, versioned config: model, system prompt, tools |
| **Session** | Every run | Links agent to environment; Anthropic runs the loop |

## Critical: stream-first pattern

Open the stream **before** sending the message — otherwise you'll miss events that arrive before your reader starts. In the CLI this means opening the stream on a file descriptor first, then sending:

```bash
# Open the stream FIRST
exec {stream}< <(ant beta:sessions:events stream --session-id "$SID" --format yaml)
# Then send — this triggers the agent loop
ant beta:sessions:events send --session-id "$SID" <<'YAML'
events:
  - type: user.message
    content:
      - type: text
        text: Hello
YAML
# Then read from $stream
```

## Files

- `hello_world_cli.ipynb` — step-by-step notebook running `ant` commands cell by cell
- `environment.yaml` / `agent.md` — written by the notebook; the version-controlled resource definitions
- `claude-lock.json` — written by `ant apply`; gitignored here (see above)

## Setup

**1. Install the CLI** — version 1.40 or later, for `ant apply` (pick one):

```bash
brew install anthropics/tap/ant                          # macOS
go install github.com/anthropics/anthropic-cli/cmd/ant@latest   # from source
# Linux/WSL: download a release from github.com/anthropics/anthropic-cli/releases
```

**2. Authenticate.** The CLI reads credentials the same way the SDKs do. Easiest is to have `ANTHROPIC_API_KEY` set in the environment (the same key the Python notebooks read from the root `.env`):

```bash
cp ../.env.example ../.env
# edit ../.env and add your ANTHROPIC_API_KEY, then export it for the CLI:
export ANTHROPIC_API_KEY=sk-ant-...
```

Or run `ant auth login` once to store an OAuth profile. `ant auth status` shows which credential source won.

## Useful commands

```bash
ant apply agent.md environment.yaml --dry-run          # show the plan, change nothing
ant beta:agents list --transform '{id,name,model}' --format jsonl
ant beta:agents:versions list --agent-id "$AGENT_ID"   # version history
ant beta:sessions:events list --session-id "$SID" --transform 'content.0.text' -r
ant --help                                             # explore
```
