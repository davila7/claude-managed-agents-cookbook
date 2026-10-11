# 04 - Files

Move files into the agent's sandbox and get its work back out.

## What you'll learn

- Upload files with the Files API and mount them into a session through `resources`
- Where mounted files land (`/mnt/session/uploads/...`) and that they're read-only
- Mount another file into a running session with `sessions.resources.add()`
- Collect what the agent writes to `/mnt/session/outputs/`: list by session scope, then download

## The flow

```
files.upload(file=path)                         →  file_id
sessions.create(resources=[{"type": "file",     →  read-only copy at /mnt/session/uploads/<mount_path>
    "file_id": ..., "mount_path": "/contracts/x.md"}])
sessions.resources.add(session_id, ...)          →  mount more files mid-session
agent writes /mnt/session/outputs/report.md      →  uploaded to the Files API, scoped to the session
beta.files.list(scope_id=session.id, betas=["managed-agents-2026-04-01"])
files.download(file_id).write_to_file(...)
```

## Gotchas

| | |
|---|---|
| Only `/mnt/session/outputs/` comes back | Tell the agent in its system prompt where to write deliverables |
| Outputs can lag the idle | They may appear a few seconds after `session.status_idle`; poll the list briefly |
| The session's file list includes your mounted inputs | Their session copies report `downloadable: False`; skip them |
| Filenames matter | The agent sees the uploaded filename; without a `mount_path` the file is named after its `file_id` |
| Uploads persist | Originals count against your storage until you delete them; session copies don't |

## Files

- `files.ipynb` — a contract analyst reviews the supplier contracts in [`../sample-data/contracts/`](../sample-data/contracts/), writes `summary.csv` and `report.md`, then folds in a contract added mid-session
- `outputs/` — written by the notebook when it downloads the results (gitignored)

## Setup

Shared `.env` at the project root. Dependencies are installed in the first cell. Run cells top to bottom.

CLI version: [`04-files-cli/`](../04-files-cli/).
