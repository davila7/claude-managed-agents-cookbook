# Claude Managed Agents Cookbook

Practical Jupyter notebooks for learning [Claude Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview) — from a minimal hello-world up to dynamic workflows, where the agent writes a program that runs many agents in parallel.

## Structure

| Folder | What you'll learn |
|--------|------------------|
| [`01-basics/`](01-basics/) | Core flow with the Python SDK: create environment → agent → session → stream events |
| [`01-basics-cli/`](01-basics-cli/) | The same core flow with the `ant` CLI: version-controlled YAML + stream-first from the terminal |
| [`02-multi-turn/`](02-multi-turn/) | Keep a session alive across turns with the Python SDK; context preserved automatically |
| [`02-multi-turn-cli/`](02-multi-turn-cli/) | The same multi-turn flow with the `ant` CLI: one reused session + a `chat()` helper |

### Roadmap

Each topic ships as an SDK module and a `-cli` companion. The path leads up to dynamic workflows, introducing one prerequisite at a time:

| Module | What it adds |
|--------|--------------|
| `03-tools-and-permissions` | Custom tools and permission policies — answering the agent mid-turn (`requires_action`) |
| `04-files` | Mount files into the sandbox and download what the agent produces |
| `05-session-control` | Steer and interrupt, reconnect and list past events, session budgets and usage |
| `06-subagents` | The `multiagent` block: delegate to other agents in session threads |
| `07-dynamic-workflows` | The agent writes a workflow that runs many agents in phases; follow the run on the stream |

## Setup

**1. Clone and configure your API key**

```bash
cp .env.example .env
# edit .env and set ANTHROPIC_API_KEY=sk-ant-...
```

**2. Install dependencies**

```bash
pip install -U "anthropic>=1.13" python-dotenv jupyter
```

> Each notebook also runs `%pip install -U anthropic python-dotenv` in its first cell, so the SDK stays up to date when you execute it.

> **For the `-cli` notebooks** you also need Anthropic's `ant` CLI, **version 1.40 or later** (for `ant apply`): `brew install anthropics/tap/ant` (macOS) or `go install github.com/anthropics/anthropic-cli/cmd/ant@latest`. If you install it while a Jupyter kernel is already running, **restart the kernel** so the new binary is on `PATH`. The CLI setup cell loads your key from `.env` and clears Jupyter's `FORCE_COLOR` (which would otherwise corrupt `ant`'s JSON/YAML output).

> **Troubleshooting:** if every SDK call fails with `APIConnectionError: Connection error.`, check for an old `brotli` package. The SDK's HTTP client needs Brotli 1.2 or later to decode compressed responses, and the real `TypeError` is hidden behind the connection error. Fix it with `pip install -U "brotli>=1.2"`, then restart the kernel.

**3. Open a notebook**

```bash
jupyter lab
# or open the .ipynb file directly in VS Code
```

## Prerequisites

- Python 3.9+
- Jupyter (JupyterLab, VS Code, or similar)
- An [Anthropic API key](https://console.anthropic.com/) with Managed Agents access

## Core concepts

| Object | Lifecycle | Purpose |
|--------|-----------|---------|
| **Environment** | Create once | Sandboxed container where tools run (bash, files, code execution) |
| **Agent** | Create once | Persisted, versioned config: model, system prompt, tools |
| **Session** | Per run | Links agent + environment; Anthropic runs the agent loop |

**In production:** persist `environment.id` and `agent.id` — don't recreate them on every run.

## Stream-first pattern

Always open the event stream **before** sending a message, or you'll miss early events:

```python
with client.beta.sessions.events.stream(session.id) as stream:
    client.beta.sessions.events.send(session_id=session.id, events=[...])
    for event in stream:
        ...
```
