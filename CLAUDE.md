# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

A cookbook of Jupyter notebooks for learning Claude Managed Agents. Each numbered folder covers a specific topic, progressing from basics to advanced patterns. Notebooks are designed to be executed cell by cell with explanations between code sections.

## Structure convention

- Folders are numbered (`01-basics/`, `02-multi-turn/`, etc.) and self-contained
- Each folder has a `README.md` and one or more `.ipynb` notebooks
- A single `.env` at the **project root** is shared by all notebooks — never put `.env` inside a subfolder

## Environment setup

```bash
cp .env.example .env   # then add ANTHROPIC_API_KEY
pip install anthropic python-dotenv jupyter
```

Notebooks install their own deps in the first cell via `%pip install -q ...`.

## Key patterns enforced across all notebooks

- **stream-first**: always open `client.beta.sessions.events.stream(session_id)` before calling `events.send()` — missing events otherwise
- **create once, reuse IDs**: `environments.create()` and `agents.create()` are setup steps, not per-run steps; production code would persist these IDs
- **version pinning**: sessions reference `agent.id` + `agent.version` explicitly
- **termination check**: event loops break on both `session.status_idle` and `session.status_terminated`
- **dotenv**: notebooks load `.env` via `find_dotenv()` so the root file is found regardless of notebook location

## Adding a new section

1. Create a new numbered folder (e.g., `02-multi-turn/`)
2. Add a `README.md` explaining what the section covers and the concepts introduced
3. Write the notebook with alternating markdown (concept explanation) and code cells
4. Update the structure table in the root `README.md`
5. The last markdown cell of each notebook should summarize the flow and point to the next section

## SDK and model defaults

- SDK: `anthropic` Python package (`client.beta.agents`, `client.beta.environments`, `client.beta.sessions`)
- Default model: `claude-opus-4-7`
- Thinking: `thinking: {"type": "adaptive"}` for complex tasks
- Beta header is set automatically by the SDK for all `client.beta.*` calls
