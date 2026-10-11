# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

A cookbook of Jupyter notebooks for learning Claude Managed Agents. Each numbered folder covers a specific topic, progressing from basics to advanced patterns. Notebooks are designed to be executed cell by cell with explanations between code sections.

## Structure convention

- Folders are numbered (`01-basics/`, `02-multi-turn/`, etc.) and self-contained
- Each folder has a `README.md` and one or more `.ipynb` notebooks
- **Every topic ships in two flavors: SDK first, then CLI.** For topic `NN-x`, author the Python-SDK module `NN-x/` first, then a companion `NN-x-cli/` that teaches the *same* flow with the `ant` CLI (version-controlled `environment.yaml` + `agent.md` reconciled with `ant apply --yes --lock-file claude-lock.json`, stream-first from the shell). The CLI README links back to the SDK one and mirrors its concepts.
  - Precedent: every module from `01-basics/` + `01-basics-cli/` through `07-dynamic-workflows/` + `07-dynamic-workflows-cli/`
  - CLI notebooks run `ant` through `subprocess` for streaming (`--format json`, parsed per top-level `}`; `jsonl` buffers when piped) and set `TERM=dumb` in the setup cell
- Shared fictitious datasets live in `sample-data/` (e.g. `sample-data/contracts/`, used by modules 04, 06, 07)
- A single `.env` at the **project root** is shared by all notebooks — never put `.env` inside a subfolder

## Environment setup

```bash
cp .env.example .env   # then add ANTHROPIC_API_KEY
pip install "anthropic>=1.13" python-dotenv jupyter
brew install anthropics/tap/ant   # CLI notebooks need ant >= 1.40 (`ant apply`)
```

Notebooks install their own deps in the first cell via `%pip install -q ...`.

## Key patterns enforced across all notebooks

- **stream-first**: always open `client.beta.sessions.events.stream(session_id)` before calling `events.send()` — missing events otherwise
- **create once, reuse IDs**: `environments.create()` and `agents.create()` are setup steps, not per-run steps; production code would persist these IDs
- **version pinning**: sessions reference `agent.id` + `agent.version` explicitly
- **termination check**: event loops break on `session.status_terminated`, and on `session.status_idle` only when its `stop_reason` says the turn is over (`end_turn`); `requires_action` means the client must answer (tool result / confirmation) and keep reading
- **lockfile stays local**: `claude-lock.json` holds org/workspace ids and is gitignored; CLI notebooks read resource ids from it
- **dotenv**: notebooks load `.env` via `find_dotenv()` so the root file is found regardless of notebook location

## Adding a new section

1. Create a new numbered folder (e.g., `02-multi-turn/`)
2. Add a `README.md` explaining what the section covers and the concepts introduced
3. Write the notebook with alternating markdown (concept explanation) and code cells
4. Update the structure table in the root `README.md`
5. The last markdown cell of each notebook should summarize the flow and point to the next section

## SDK and model defaults

- SDK: `anthropic` Python package (`client.beta.agents`, `client.beta.environments`, `client.beta.sessions`)
- SDK version: `anthropic>=1.13` (needed for `multiagent_20261001`, workflow-run events, session budgets)
- Default model: `claude-opus-5-5`
- Effort: set via the object form of `model`, e.g. `{"id": "claude-opus-5-5", "effort": "high"}`
- Docs: https://platform.claude.com/docs/en/managed-agents/overview
- Beta header is set automatically by the SDK for all `client.beta.*` calls
