# Claude Managed Agents Cookbook

Practical Jupyter notebooks for learning [Claude Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview) — from a minimal hello-world up to dynamic workflows, where the agent writes a program that runs many agents in parallel.

> The notebooks call the real API, so running them costs money. Modules 01–05 cost cents each; 06 and 07 run several agents, and each session sets a budget ($3 and $5) as a hard cap.

## Structure

| Folder | What you'll learn |
|--------|------------------|
| [`01-basics/`](01-basics/) | Core flow with the Python SDK: create environment → agent → session → stream events |
| [`01-basics-cli/`](01-basics-cli/) | The same core flow with the `ant` CLI: version-controlled files + `ant apply`, stream-first from the terminal |
| [`02-multi-turn/`](02-multi-turn/) | Keep a session alive across turns with the Python SDK; context preserved automatically |
| [`02-multi-turn-cli/`](02-multi-turn-cli/) | The same multi-turn flow with the `ant` CLI: one reused session + a `chat()` helper |
| [`03-tools-and-permissions/`](03-tools-and-permissions/) | Custom tools and permission policies: answer the agent mid-turn, read `stop_reason` |
| [`03-tools-and-permissions-cli/`](03-tools-and-permissions-cli/) | The same with `ant`: tool config in `agent.md` frontmatter, updates through `ant apply` |
| [`04-files/`](04-files/) | Upload and mount files into the sandbox, add files mid-session, download the agent's outputs |
| [`04-files-cli/`](04-files-cli/) | The same with `ant files` and `ant beta:sessions:resources` |
| [`05-session-control/`](05-session-control/) | Budgets and usage, `system.message`, interrupt + redirect, reconnect without missing events, resume at the cap |
| [`05-session-control-cli/`](05-session-control-cli/) | The same with `ant beta:sessions update / retrieve / archive` and `ant ... events list` |
| [`06-subagents/`](06-subagents/) | The `multiagent` block: a lead delegates to specialist subagents, each in its own session thread |
| [`06-subagents-cli/`](06-subagents-cli/) | The same with `ant`: subagents referenced by file path, `ant beta:sessions:threads` |
| [`07-dynamic-workflows/`](07-dynamic-workflows/) | The agent writes a workflow that runs many agents in phases; follow the run to the end |
| [`07-dynamic-workflows-cli/`](07-dynamic-workflows-cli/) | The same with `ant`: workflows enabled in `agent.md`, the run followed from the shell |

### Learning path

Each topic ships as an SDK module and a `-cli` companion. The modules build on each other, up to dynamic workflows:

```
01 basics → 02 multi-turn → 03 tools & permissions → 04 files → 05 session control → 06 subagents → 07 dynamic workflows
```

Shared fictitious data used by several modules lives in [`sample-data/`](sample-data/).

**Step-by-step guide (PDF):** [English](docs/Claude-Managed-Agents-Guide-EN.pdf) · [Español](docs/Claude-Managed-Agents-Guia-ES.pdf). It covers what Managed Agents is, VS Code setup, and what to watch on platform.claude.com while you run each module.

## Setup

**1. Clone and configure your API key**

```bash
cp .env.example .env
# edit .env and set ANTHROPIC_API_KEY=sk-ant-...
```

**2. Create a virtual environment and install dependencies**

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -U "anthropic>=1.13" python-dotenv jupyter
```

In VS Code, pick `.venv` as the notebook kernel (**Select Kernel → Python Environments → .venv**).

> Each notebook also runs `%pip install -U anthropic python-dotenv` in its first cell, so the SDK stays up to date when you execute it.

> **For the `-cli` notebooks** you also need Anthropic's `ant` CLI, **version 1.40 or later** (for `ant apply`): `brew install anthropics/tap/ant` (macOS) or `go install github.com/anthropics/anthropic-cli/cmd/ant@latest`. If you install it while a Jupyter kernel is already running, **restart the kernel** so the new binary is on `PATH`. The CLI setup cell loads your key from `.env` and clears Jupyter's `FORCE_COLOR` (which would otherwise corrupt `ant`'s JSON/YAML output).

> **Troubleshooting:** if every SDK call fails with `APIConnectionError: Connection error.`, check for an old `brotli` package. The SDK's HTTP client needs Brotli 1.2 or later to decode compressed responses, and the real `TypeError` is hidden behind the connection error. Fix it with `pip install -U "brotli>=1.2"`, then restart the kernel.

**3. Open a notebook**

```bash
jupyter lab
# or open the .ipynb file directly in VS Code
```

## Prerequisites

- Python 3.10+ (the notebooks use `match` statements)
- Jupyter (JupyterLab, VS Code, or similar)
- A [Claude API key](https://platform.claude.com/settings/keys) (Managed Agents is enabled by default for API accounts)

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
