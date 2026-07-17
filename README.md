# n8n-workflows

Shared workflow store for n8n + OpenCode automation.

## Purpose
Central place for n8n workflows used to orchestrate OpenCode tasks:
- Code review loops with quality gates (PASS → done, FAIL → re-run OpenCode)
- Sequential task packs (work packages processed one after another)
- Local repo scans / QoL / feature pipelines

## Architecture
- **Linux server** (`vm-cf55bc17`): n8n runs natively via systemd (localhost:5678), reached from the Mac over an SSH tunnel. OpenCode runs locally for Linux-repo work.
- **Windows machine**: separate local n8n + OpenCode instance. Reaches this repo over the internet (no VPN needed).
- This repository is the **only shared layer** between the two machines. Each machine keeps its own repos/state locally; only workflow definitions are synced here.

## Layout
```
workflows/   n8n workflow JSON exports (general)
opencode/    OpenCode-specific pipelines (review-loop, task packs)
```

## Sync workflow
1. Edit workflow in n8n UI → Export as JSON → save into the matching folder
2. `git add . && git commit -m "..." && git push`
3. On the other machine: `git pull` → Import JSON into n8n

## Repo URL
https://github.com/MichaOro/n8n-workflows
