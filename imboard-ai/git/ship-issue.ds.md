---dossier
{
  "dossier_schema_version": "1.0.0",
  "title": "Ship Issue — Commit, PR, Merge, Teardown",
  "version": "1.1.0",
  "status": "Stable",
  "objective": "Commit changes, push, create a PR, wait for CI, merge, and clean up the worktree",
  "category": [
    "development"
  ],
  "tags": [
    "issue",
    "workflow",
    "git",
    "github",
    "ship",
    "pr",
    "merge"
  ],
  "risk_level": "medium",
  "risk_factors": [
    "modifies_files",
    "network_access",
    "creates_pull_request",
    "merges_code"
  ],
  "destructive_operations": [
    "Pushes branch to remote",
    "Creates and merges pull request",
    "Deletes branch after merge"
  ],
  "inputs": {
    "required": [
      {
        "name": "issue_number",
        "description": "GitHub issue number",
        "type": "number"
      }
    ],
    "optional": [
      {
        "name": "base_branch",
        "description": "Target branch for the PR",
        "type": "string",
        "default": "main"
      },
      {
        "name": "review_escalated",
        "description": "List of escalated review findings to create GH issues for",
        "type": "array",
        "default": []
      },
      {
        "name": "worktree_path",
        "description": "Path of the worktree to clean up after merge",
        "type": "string",
        "default": ""
      },
      {
        "name": "original_dir",
        "description": "Directory to return to after teardown",
        "type": "string",
        "default": ""
      },
      {
        "name": "pool_claimed",
        "description": "Whether the worktree was claimed from the pool (affects cleanup)",
        "type": "boolean",
        "default": false
      }
    ]
  },
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "name": "ship-issue",
  "checksum": {
    "algorithm": "sha256",
    "hash": "3a075ce215f7a9ade01e017c2777b1a0e8b81a5621d72ef2b9fb86874bdbb012"
  }
}
---

# Ship Issue — Commit, PR, Merge, Teardown

## Objective

Ship the implementation: commit, push, create PR, wait for CI, merge, and clean up.

## Prerequisites

- All code changes are ready (implemented, tested, reviewed)
- You are in the worktree/working directory with uncommitted changes
- GitHub CLI (gh) is installed and authenticated
- You have push access

## Actions to Perform

### Step 1: Commit

1. Stage relevant files (never `.env`, credentials, secrets)
2. Commit with conventional commits format:
   ```
   feat|fix|chore: <description>

   Closes #<issue_number>

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```

### Step 2: Push

```bash
git push -u origin <branch-name>
```

### Step 3: Create PR

Create the PR targeting `base_branch`:

```bash
gh pr create --base <base_branch> --title "<short title>" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points>

Closes #<issue_number>

## Test plan
- <how to verify>

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### Step 4: Create GH Issues for Escalated Findings

After the PR is created, handle any escalated findings from the review phase.

**Consolidation rule**: Group all escalated findings from the same review category (DRY, Security, Supportability, Maintainability, Documentation) into a **single GH issue per category**. This means at most 5 issues total (one per review agent), but typically 0.

Most PRs should have **zero** escalated issues. If there are more than 2, re-evaluate.

For each category with escalated findings:

```bash
gh issue create --title "<Category>: escalated findings from PR #<pr-number>" --label "review" --body "$(cat <<'EOF'
**Source**: PR #<pr-number>, found during <Category> review of <branch>

### Findings

1. **<file>:<lines>** — <description>
   **Why escalated**: <why this needs human judgment>
EOF
)"
```

If zero escalated findings (common case), create no GH issues.

### Step 5: Wait for CI — stable-confirmation gate

**Do NOT merge until checks are CONFIRMED green, and guard against false positives.**
The GitHub check-runs API (and `gh pr checks --watch`) intermittently report a phantom
`completed`/`success` for a job that is still running, and can momentarily show all-pass
*before* a slow required job (e.g. the backend integration suite) has even registered.
Merging on a single transient read ships an unverified build. This has bitten real runs —
treat one green read as **unconfirmed**.

The merge gate has two conditions, **both confirmed on two consecutive polls ≥20s apart**:

1. `mergeStateStatus` is `CLEAN` (via `gh pr view <pr-number> --json mergeStateStatus`).
2. `gh pr checks <pr-number>` shows **zero** checks still `pending`/`in_progress` and
   **zero** in any failing state.

Poll loop (do not merge until it exits with `stable=2`):

```bash
stable=0
while [ "$stable" -lt 2 ]; do
  mss=$(gh pr view <pr-number> --json mergeStateStatus --jq '.mergeStateStatus')
  checks=$(gh pr checks <pr-number> 2>/dev/null)
  pend=$(printf '%s\n' "$checks" | grep -ciE 'pending|in_progress')
  fail=$(printf '%s\n' "$checks" | grep -ciE 'fail|failure|error|cancel|timed_out')
  echo "mss=$mss pending=$pend failing=$fail stable=$stable"
  if [ "$fail" -gt 0 ]; then break; fi                 # real or flaky failure -> Step 6
  if [ "$mss" = "CLEAN" ] && [ "$pend" -eq 0 ]; then
    stable=$((stable + 1))                              # one confirmed-clean read
  else
    stable=0                                            # reset on any not-yet-ready read
  fi
  [ "$stable" -lt 2 ] && sleep 25
done
```

- `stable` reaching `2` = two consecutive fully-green reads ⇒ proceed to Step 7.
- `failing > 0` ⇒ go to Step 6.
- `mergeStateStatus` values other than `CLEAN` (`UNSTABLE`, `BLOCKED`, `BEHIND`, `UNKNOWN`)
  reset the counter — `UNSTABLE`/`UNKNOWN` usually just means not every check has reported
  yet; keep polling, do not treat it as a failure on its own.

**Known transient false-blocks (re-run once, don't escalate):** if a required check fails
but the signature is a known-flaky/environmental one *unrelated to the diff* — a codegen
race (e.g. a docs/`.source` "not a module" / missing-property typecheck on a PR that
touches no docs), or a freshly-published dependency advisory the PR never introduced
(OSV-Scanner on a transitive dep) — re-run that job once (`gh run rerun <run-id> --failed`)
before treating it as a real Step-6 failure. If a security advisory is genuinely on `main`
too, the fix is a separate dependency-bump PR, not this one.

### Step 6: Handle CI Failures (max 2 attempts)

If CI fails:

1. Identify which job failed:
   ```bash
   gh pr checks <pr-number>
   ```
2. Get the failed run ID and view logs:
   ```bash
   gh run view <run-id> --log-failed
   ```
3. Read the failure output, identify root cause, fix the code
4. Run the failing command locally first to confirm the fix
5. Commit and push:
   ```bash
   git add <files> && git commit -m "fix: CI failure — <what was wrong>

   Co-Authored-By: Claude <noreply@anthropic.com>"
   git push
   ```
6. Wait for CI again (go back to Step 5)
7. If CI fails after 2 fix attempts, ask the user — do not merge a red build

### Step 7: Merge

**Only after the Step 5 gate exited with `stable=2`.** Re-confirm with one final read
immediately before merging — `gh pr view <pr-number> --json mergeStateStatus` must still
be `CLEAN`. If it regressed to `UNSTABLE`/`BLOCKED` (a check re-queued or a new push
landed), return to Step 5; never merge on a stale green.

All checks confirmed green — merge:

```bash
gh pr merge <pr-number> --squash
```

Do NOT use `--delete-branch` — it fails from worktrees. Branch cleanup happens in Step 8.

Clean up issue labels:

```bash
gh issue edit <issue_number> --remove-label "in-progress"
```

### Step 8: Teardown

**Prerequisite: Step 7 (Merge) must be complete.** Do not tear down before merge.

1. `cd` back to `original_dir` (if provided)
2. **Try to return the worktree to the pool** (if `pool_claimed` is true):
   ```bash
   npx worktree-pool return --path <worktree_path> 2>/dev/null
   ```
   - If succeeds: worktree recycled for reuse. Skip steps 3-5.
   - If fails: continue with manual cleanup.
3. Remove the worktree:
   ```bash
   git worktree remove <worktree_path>
   ```
4. Delete local branch if it still exists:
   ```bash
   git branch -d <branch-name> 2>/dev/null || git branch -D <branch-name>
   ```
5. Clean up remote branch if it still exists:
   ```bash
   git push origin --delete <branch-name> 2>/dev/null || true
   ```

## Output

- `pr_number`: the created PR number
- `pr_url`: the PR URL
- `merge_status`: merged | failed
- `target_branch`: the branch merged into
- `escalated_issues`: list of created GH issue numbers (may be empty)
- `cleanup`: pool_returned | worktree_removed | skipped

## Validation

- [ ] Changes committed with conventional commits format
- [ ] Branch pushed to remote
- [ ] PR created targeting correct base_branch
- [ ] Escalated findings consolidated per review category (typically 0)
- [ ] CI passed (or failures fixed within 2 attempts)
- [ ] CI confirmed green on two consecutive stable polls — not a single transient success
- [ ] PR merged (squash)
- [ ] in-progress label removed
- [ ] Worktree returned to pool or removed
- [ ] Returned to original directory

## Troubleshooting

**CI fails after fixes**: Ask user — may be infrastructure issue

**Phantom success / flaky check status**: the checks API can report a transient
`success` while a required job is still running, or all-pass before a slow job registers.
Never merge on one read — require two consecutive `CLEAN` + zero-pending polls (Step 5).
Don't stall passively either: drive the Step 5 loop to `stable=2`, then merge.

**Merge conflicts**: Ask user — needs human judgment

**`--delete-branch` fails in worktree**: Expected — don't use it. Clean up in Step 8.

**Pool return fails**: Not an error — fall back to manual worktree remove.
