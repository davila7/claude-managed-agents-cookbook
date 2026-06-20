# 01 - Basics (CLI)

The same minimal end-to-end example as [`01-basics/`](../01-basics/), driven from the terminal with Anthropic's **`ant`** CLI instead of the Python SDK. One section, two flavors: pick the SDK for application code, the CLI for ad-hoc control-plane work and version-controlled YAML.

## What you'll build

A complete agent flow from scratch:

```
ant beta:environments create < env.yaml   →  store the environment id
ant beta:agents create < agent.yaml        →  store agent id + version
ant beta:sessions create --agent ...        →  new session per run
ant beta:sessions:events stream             →  open stream FIRST
ant beta:sessions:events send               →  send message, triggers agent loop
read loop                                   →  handle responses until idle
```

## CLI for the control plane, SDK for the data plane

| | Control plane → `ant` | Data plane → SDK / CLI |
|---|---|---|
| Resources | agents, environments | sessions, events |
| Cadence | Once per deploy / ad-hoc | Every task / every turn |
| Lives in | `*.yaml` in your repo + CI | Application code |

Agents and environments are relatively static — define them as YAML, check them into your repo, apply with `ant beta:agents create|update`. Sessions are dynamic — created per run and streamed.

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
- `env.yaml` / `agent.yaml` — written by the notebook; the version-controlled resource definitions

## Setup

**1. Install the CLI** (pick one):

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
ant beta:agents list --transform '{id,name,model}' --format jsonl
ant beta:sessions:events list --session-id "$SID" --transform 'content.0.text' -r
ant beta:agents update --agent-id "$AGENT_ID" --version N < agent.yaml   # bump version
ant --help                                                              # explore
```
