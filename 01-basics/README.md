# 01 - Basics

Minimal end-to-end example for Claude Managed Agents.

## What you'll build

A complete agent flow from scratch:

```
environments.create()  →  store environment.id
agents.create()        →  store agent.id + agent.version
sessions.create()      →  new session per run
events.stream()        →  open stream FIRST
events.send()          →  send message, triggers agent loop
for event in stream    →  handle responses until status_idle
```

## Core concepts

| Object | Created | Purpose |
|--------|---------|---------|
| **Environment** | Once | Sandboxed container where tools run (bash, file ops, code execution) |
| **Agent** | Once | Persisted, versioned config: model, system prompt, tools |
| **Session** | Every run | Links agent to environment; Anthropic runs the loop |

## Critical: stream-first pattern

Open the stream **before** calling `events.send()` — otherwise you'll miss events that arrive before your iterator starts:

```python
with client.beta.sessions.events.stream(session.id) as stream:
    client.beta.sessions.events.send(...)   # send AFTER opening stream
    for event in stream:
        ...
```

## Files

- `hello_world.ipynb` — step-by-step notebook with explanations for each section

## Setup

The `.env` file lives at the **project root** (shared by all notebooks):

```bash
cp ../.env.example ../.env
# edit ../.env and add your ANTHROPIC_API_KEY
```

Dependencies are installed in the first cell of the notebook (`%pip install -U anthropic python-dotenv`). No manual install needed — just run the cells in order.
