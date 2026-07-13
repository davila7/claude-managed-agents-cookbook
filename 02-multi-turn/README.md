# 02 - Multi-Turn Conversations

Keep a session alive across multiple exchanges and let the agent build on prior context.

## What you'll learn

- A session stays alive after `status_idle` — you can send another message immediately
- Conversation history is maintained server-side; you never resend previous turns
- How to wrap the stream-first pattern in a reusable helper
- An optional interactive REPL pattern for live chat

## The pattern

```python
session = client.beta.sessions.create(...)   # one session per conversation

# Turn 1
with client.beta.sessions.events.stream(session.id) as stream:
    client.beta.sessions.events.send(session_id=session.id, events=[msg_1])
    for event in stream:
        if event.type == "session.status_idle":
            break   # agent is ready for the next message

# Turn 2 — same session, context preserved automatically
with client.beta.sessions.events.stream(session.id) as stream:
    client.beta.sessions.events.send(session_id=session.id, events=[msg_2])
    for event in stream:
        if event.type == "session.status_idle":
            break
```

## Session status reference

| Status | Meaning |
|---|---|
| `session.status_idle` | Agent finished its turn — send another message |
| `session.status_terminated` | Session closed — create a new one |

## Files

- `multi_turn.ipynb` — step-by-step notebook with the chat helper and interactive REPL

## Setup

Shared `.env` at the project root (same as module 01):

```bash
cp ../.env.example ../.env
# add your ANTHROPIC_API_KEY
```

Dependencies are installed in the first cell. Run cells top to bottom.
