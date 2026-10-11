# 04 - Files (CLI)

The same flow as [`04-files/`](../04-files/), driven with the **`ant`** CLI: upload contracts, mount them into a session, let the agent write a report, add one more file mid-session, and download the results.

## Commands

```bash
ant files upload --file contract.md --transform id -r          # -> file_id

ant beta:sessions create --agent "$AGENT" --environment-id "$ENV" \
  --resource '{"type": "file", "file_id": "file_...", "mount_path": "/contracts/contract.md"}'
  # repeat --resource once per file; lands at /mnt/session/uploads/contracts/contract.md

ant beta:sessions:resources add --session-id "$SID" --type file \
  --file-id "$FILE_ID" --mount-path /contracts/late.md          # mount into a running session
ant beta:sessions:resources list --session-id "$SID"

ant beta:files list --scope-id "$SID" --beta managed-agents-2026-04-01   # outputs (+ input copies)
ant files download --file-id "$FILE_ID" --output outputs/report.md
ant files delete --file-id "$FILE_ID"
```

The scoped file list also includes the session copies of your mounted inputs; they report `downloadable: false`. Outputs can appear a few seconds after the session goes idle.

## Files

- `files_cli.ipynb` — step-by-step notebook
- `environment.yaml` / `agent.md` — written by the notebook
- `claude-lock.json`, `outputs/` — written when you run it; gitignored

## Setup

Requires the `ant` CLI (>= 1.40) installed and authenticated — see [`01-basics-cli/`](../01-basics-cli/#setup). Run cells top to bottom.
