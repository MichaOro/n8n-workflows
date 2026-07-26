# OpenCode pipelines

OpenCode-specific n8n workflows for automated code review and task pack execution.

## Workflows

### `review-loop.json` — Automated Code Review Loop

Runs OpenCode with a review prompt, evaluates quality gate, retries on FAIL with feedback.

**Features:**
- Configurable review prompt, quality threshold (0-100), max retries
- Quality gate: PASS if `quality_score >= threshold`, else FAIL
- On FAIL: constructs feedback prompt with issues, retries up to `max_retries` via `SplitInBatches`
- Final notification with PASS/FAIL result and summary

**Configuration (Manual Trigger input):**
```json
{
  "repo_path": "={{$env.REPO_ROOT}}/vCardOrologas",
  "review_prompt": "Review the code changes in this PR. Check for: correctness, security, performance, style. Output JSON: {quality_score: 0-100, issues: string[], summary: string}",
  "model": "openrouter/nvidia/nemotron-3-ultra-550b-a55b:free",
  "quality_threshold": 70,
  "max_retries": 3
}
```

**OpenCode Output Format Expected (`--format json`):**
```json
{
  "exitCode": 0,
  "stdout": "...",
  "stderr": "",
  "result": {
    "quality_score": 85,
    "issues": ["Issue 1", "Issue 2"],
    "summary": "Review summary text"
  }
}
```

**Requirements:**
- `OPENROUTER_API_KEY` environment variable on n8n host
- OpenCode CLI installed and in PATH
- Workflow tagged `synced` for Git sync
- `$env.REVIEW_WEBHOOK_URL` for notifications (optional)

---

### `task-pack.json` — Sequential Task Pack Pipeline

Processes a work package (array of tasks) sequentially, each using OpenCode, with result aggregation.

**Features:**
- Accepts task pack array via Manual Trigger
- Sequential execution via `SplitInBatches` (batchSize: 1)
- Stops on first failure if `continue_on_failure: false`
- Aggregates all results via `$workflow.staticData`
- Final notification with summary (success/failure counts, per-task details)

**Input Format (Manual Trigger JSON):**
```json
{
  "task_pack": [
    {"name": "task-1", "prompt": "Refactor auth module", "repo_path": "={{$env.REPO_ROOT}}/vCardOrologas"},
    {"name": "task-2", "prompt": "Add unit tests for auth", "repo_path": "={{$env.REPO_ROOT}}/vCardOrologas"},
    {"name": "task-3", "prompt": "Update documentation", "repo_path": "={{$env.REPO_ROOT}}/vCardOrologas"}
  ],
  "model": "openrouter/nvidia/nemotron-3-ultra-550b-a55b:free",
  "continue_on_failure": false
}
```

**Output (Notification):**
```
Task Pack Execution Complete
===============================
Total Tasks: 3
Successful: 2
Failed: 1
Overall: PARTIAL FAILURE

Task Details:
1. task-1: ✅
   Exit Code: 0
   Output: Refactored auth module...
   Error: 

2. task-2: ❌
   Exit Code: 1
   Output: 
   Error: Test framework not configured

3. task-3: ✅
   Exit Code: 0
   Output: Updated README...
   Error: 
```

**Requirements:**
- `OPENROUTER_API_KEY` environment variable on n8n host
- OpenCode CLI installed and in PATH
- `$env.REPO_ROOT` environment variable set per machine (Linux: `/home/orologas`, Windows: `C:\Users\micha`)
- Workflow tagged `synced` for Git sync
- `$env.TASKPACK_WEBHOOK_URL` for notifications (optional)

---

### `mini-pocket.json` — OpenCode Mini Pocket

Simple test workflow to verify OpenCode CLI execution.

---

## Common Requirements

| Requirement | Details |
|-------------|---------|
| OpenCode CLI | Installed on n8n host machine |
| OPENROUTER_API_KEY | Set in n8n host environment |
| Model | Default: `openrouter/nvidia/nemotron-3-ultra-550b-a55b:free` |
| Git Sync | All workflows tagged `synced` for bidirectional sync |
| n8n Version | Tested on n8n 1.x |

---

## Usage

1. Import workflow JSON into n8n (Workflows → Import)
2. Configure environment variables on host machine:
   - `OPENROUTER_API_KEY` — OpenRouter API key for OpenCode
   - `REPO_ROOT` — Repository root path (Linux: `/home/orologas`, Windows: `C:\Users\micha`)
   - `REVIEW_WEBHOOK_URL` — Discord/webhook URL for review notifications (optional)
   - `TASKPACK_WEBHOOK_URL` — Discord/webhook URL for task pack notifications (optional)
3. Tag workflow with `synced` in n8n UI (Tags → Add → `synced`)
4. Execute via Manual Trigger with appropriate input JSON

---

## Git Sync

All workflows are tagged `synced` for bidirectional Git synchronization per `WORK_PACKAGE_SYNC.md` (WP-3/WP-4). See repository root `README.md` for sync architecture.

Tag format in n8n workflow JSON: `[{"id": "1", "name": "synced"}]`

---

## Cross-Platform Notes

### Linux (bash)
```bash
export OPENROUTER_API_KEY="$(python3 -c "import json;print(json.load(open('/home/orologas/.local/share/opencode/auth.json'))['openrouter']['key'])")"
cd {{$json.repo_path}}
opencode run '{{$json.prompt}}' --model {{$json.model}} --format json
```

### Windows (PowerShell)
```powershell
$key = (Get-Content "$env:USERPROFILE\AppData\Local\opencode\auth.json" | ConvertFrom-Json).openrouter.key
$env:OPENROUTER_API_KEY = $key
cd {{$json.repo_path}}
opencode run '{{$json.prompt}}' --model {{$json.model}} --format json
```

---

## Testing

Use `opencode/mini-pocket.json` repository (`vCardOrologas`) as test target for both workflows.