# WP-6: Cross-Platform `sync-from-git` Workflow

## Objective
Make the `sync-from-git.json` n8n workflow run correctly on **both** the Linux server and the Windows machine without manual editing before each execution.

## Business Value
- Removes a manual gating step each time a user wants to sync workflows from Git to their local n8n instance.
- Eliminates the risk of accidentally running the wrong path on the wrong OS, which causes silent failures or a broken import.
- Unblocks automated triggers (webhook/schedule) on both machines.

## Scope
- Modify `sync-from-git.json` so the repo path is derived from an environment variable (`$env.REPO_ROOT`), consistent with `review-loop.json` and `task-pack.json`.
- Replace both occurrences of the hard-coded Linux path (`/home/orologas/n8n-workflows`).
- Replace the Linux-specific `find` command in the shell node with a cross-platform approach (Node.js Code node with `fs.readdirSync`).
- Update the `opencode/README.md` to document the change.

## Out of Scope
- Adding new sync capabilities (e.g., push-to-git, selective sync by tag).
- Changing the API-based import/update logic in the "Import/Update Workflows via API" code node.
- Adding authentication or secrets management improvements.
- Converting other workflows (`review-loop`, `task-pack`) — they already use `$env.REPO_ROOT`.
- Adding CI/CD or automated deployment.

## Requirements

### Functional
1. The workflow MUST use `$env.REPO_ROOT` to construct the working directory path, instead of a hard-coded absolute path.
2. The workflow MUST produce the same output — a list of imported/updated workflow names with status — on both platforms.
3. The file-listing step MUST work on both Windows (PowerShell) and Linux (bash) without modification.
4. The git pull step MUST execute in the correct repo directory on both platforms.

### Non-Functional
5. Zero configuration changes required on the Linux server (where `$env.REPO_ROOT` is already `/home/orologas`).
6. On Windows, only `$env.REPO_ROOT` needs to be set (per existing convention: `C:\Users\micha`).

## Assumptions
- `$env.REPO_ROOT` is set on both machines (Linux: `/home/orologas`, Windows: `C:\Users\micha`).
- The n8n host on each machine has `git` available in PATH.
- The repo is already cloned at `$env.REPO_ROOT/n8n-workflows`.
- The n8n instance is running locally on port 5678 on both machines.

## Dependencies
- **Internal:** `sync-from-git.json` workflow, `opencode/README.md`.
- **External:** n8n `$env` variable resolution (built-in), `git` CLI.

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| `find` not available on Windows | Medium | High | Replace with Node.js `fs.readdirSync` recursive walk in a Code node |
| Shell differences break git pull | Low | Medium | Use Node.js `child_process.execSync` in a Code node instead of `executeCommand` |
| Windows path separator `\` in Code node | Low | Medium | Use `path.join()` and `path.resolve()` consistently |
| `$env.REPO_ROOT` trailing slash mismatch | Low | Low | Use `path.join(process.env.REPO_ROOT, 'n8n-workflows')` to normalize path |

## Architecture Notes
- **Current design:** The `executeCommand` node runs `cd /home/orologas/n8n-workflows && git pull origin main 2>&1; echo '---FILES---'; find . -name '*.json' -not -path './.git/*' | sort`. This is Linux-only due to the `find` command.
- **Target design:** Merge the "Git Pull + List Files" and "Parse Workflow JSONs" nodes into a single Code node that:
  1. Runs `git pull origin main 2>&1` via `child_process.execSync()` in the `$env.REPO_ROOT/n8n-workflows` directory.
  2. Recursively walks the directory with `fs.readdirSync()` to discover `.json` workflow files (excluding `.git`).
  3. Reads and parses each JSON file to extract the workflow name.
  4. Outputs the same list format the downstream "Import/Update Workflows via API" node expects.
- This eliminates the platform-dependent `executeCommand` node entirely, making the workflow truly cross-platform.
- Alternatively, if keeping `executeCommand` for simplicity, use PowerShell's `Get-ChildItem` on Windows and `find` on Linux — but the Code node approach is cleaner.

## Deliverables
1. Updated `sync-from-git.json` with the cross-platform implementation.
2. Updated `opencode/README.md` documenting that `sync-from-git` now requires only `$env.REPO_ROOT`.

## Acceptance Criteria
- [ ] On Linux, executing the workflow imports/updates all workflow JSONs from the repo into n8n.
- [ ] On Windows, executing the workflow imports/updates all workflow JSONs from the repo into n8n.
- [ ] The repo path is derived from `$env.REPO_ROOT` in both the git pull directory and file-reading code.
- [ ] No hard-coded Linux or Windows paths remain in the workflow JSON.

## Test Strategy
- **Manual test on Linux:** Run the workflow on the Linux server; verify that "Import/Update" output shows `created` or `updated` for each workflow file.
- **Manual test on Windows:** Run the workflow on the Windows machine (same `sync-from-git.json`); verify identical output.
- **Edge case:** No `.json` workflow files in repo — verify the workflow does not crash and returns an empty list gracefully.
- **Edge case:** Git pull fails (no network) — verify the workflow reports the git error without crashing.

## Definition of Done
- Workflow JSON updated and committed to `opencode/sync-from-git.json`.
- `opencode/README.md` updated to reflect the cross-platform change.
- Workflow tested on at least one platform (Linux or Windows) — tag after successful execution.

## Review Checklist
- [ ] No hard-coded absolute paths remain in any node of `sync-from-git.json`.
- [ ] Node.js code uses `path.join()` and `process.env.REPO_ROOT` (not string concatenation with hard-coded separators).
- [ ] The `opencode/README.md` sync section accurately describes the new approach.
- [ ] All workflow JSON tags still include `synced`.
- [ ] The workflow is backward-compatible with previously imported workflow state in n8n (same workflow name, same node IDs).
