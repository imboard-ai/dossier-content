---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "ship-issue",
  "title": "Ship Issue — Commit, PR, Merge, Deploy, Teardown",
  "version": "1.9.1",
  "protocol_version": "1.0",
  "status": "Stable",
  "objective": "Commit changes, push, create a PR, then either drive it to a confirmed merge and deploy (attached) or park it on auto-merge and stop (detached)",
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
  "risk_level": "high",
  "risk_factors": [
    "modifies_files",
    "network_access",
    "creates_pull_request",
    "merges_code"
  ],
  "requires_approval": false,
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
      },
      {
        "name": "run_id",
        "description": "Runstate run id minted by gate-issue; pass through unchanged",
        "type": "string"
      },
      {
        "name": "ship_mode",
        "description": "attached (default) = open the PR, wait for CI, merge, confirm the deploy, and tear down in this run. detached = open the PR, park it on auto-merge, post the awaiting-merge milestone, and STOP — a later run (gate resumes at ship-teardown) finishes it.",
        "type": "string",
        "default": "attached"
      },
      {
        "name": "ac_results",
        "description": "Per-acceptance-criterion checklist from review-issue's Agent 7 (Conformance) — criterion, verdict, file:line or reason. Used to populate the PR body's Acceptance Criteria section.",
        "type": "string"
      }
    ]
  },
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "last_updated": "2026-08-24",
  "checksum": {
    "algorithm": "sha256",
    "hash": "299bfc84ef3947fc2da8c363de8b4c6900b4b184ecb4e71d7120203e8e500c80"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "Gk5jBFO4YrmlUDe0D0iP2CZD1siCSOP4gJiIq87NEvdWmNj0mL2rJ8zKls+rVLqzbs5o1POnI/FSbhzOoS1vCQ==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-08-24T14:05:31.414Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Ship Issue — Commit, PR, Merge, Teardown

## Objective

Ship the implementation: commit, push, create PR, wait for CI, merge, and clean up.

## Prerequisites

- All code changes are ready (implemented, tested, reviewed)
- review-issue found zero escalated findings. If it found any, the full-cycle
  orchestrator should already have stopped before invoking this dossier — see
  full-cycle-issue's Guiding Principle. This dossier does not create GitHub
  issues; it assumes the diff is ready to ship as-is.
- You are in the worktree/working directory with uncommitted changes
- GitHub CLI (gh) is installed and authenticated
- `ai-dossier` CLI >= 0.10.0 is installed — both ship milestones are posted through it
- You have push access

## Actions to Perform

### Step 1: Commit

0. **If `scripts/ci-parity.sh` exists in the repo, run `bash scripts/ci-parity.sh` before committing.** It is the project's own definition of what CI enforces — catching a hygiene failure here costs seconds instead of a full CI round-trip. Fix and re-run until it passes. If the script does not exist, skip this and commit as usual.
1. Stage relevant files (never `.env`, credentials, secrets)
2. Commit with conventional commits format:
   ```
   feat|fix|chore: <description>

   Closes #<issue_number>

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```

   Note: earlier phases (plan/implement/review) already pushed `wip(...)` commits to this
   branch as they ran (WIP sync rule — see full-cycle-issue's Runstate Milestones). This
   final conventional commit lands on top of them; nothing about this step changes. Step 6's
   squash-merge collapses the whole branch history — WIP commits included — into one commit
   on the base branch.

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

## Acceptance Criteria
- [x] AC1 <criterion> — <file:line>
- [ ] AC2 <criterion> — not met / unverifiable: <reason>

## Test plan
- <how to verify>

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

The Acceptance Criteria boxes come from `ac_results` (review-issue's Agent 7 output, passed through by full-cycle-issue): a checked box with `file:line` for each `met` AC, an unchecked box with the reason for `not-met`/`unverifiable`. If `ac_results` is empty (Agent 7 was skipped — no AC list existed), omit this section.

### Step 3b: Runstate Milestone (awaiting-merge)

Post this BEFORE the CI wait — it is what tells a later reader that a PR exists and the run is parked on CI, even if this session dies mid-wait. Comments are append-only: never edit or delete a prior milestone.

```bash
ai-dossier runstate post --issue <issue_number> --phase ship --status awaiting-merge --run <run_id> \
  --kv pr=<pr-number> \
  --kv head=<short sha of the pushed commit> \
  --kv ci_fix_attempts=0 \
  --next ship
```

The CLI stamps `at=` itself. `--next ship` is the one place a dossier overrides the computed `next=` — this milestone is mid-phase, so the next phase is still ship. The CLI validates phase, status, and keys and refuses a malformed milestone; never hand-write the comment instead. Values contain no spaces (use `-` or `,`); paths are absolute.

In `ship_mode=detached` this is the run's LAST milestone (Step 3c) — it is what a later gate reads to resume at `ship-teardown`.

### Step 3c: Ship Mode — attached or detached

`ship_mode` decides whether this run drives the merge or hands it off.

**`attached` (default)** — continue to Step 4 and run the phase to its end: CI wait, merge, merge confirmation, deploy confirmation, teardown, final milestone. Nothing below changes.

**`detached`** — park the PR and end the run here:

1. Hand the merge to the watcher / merge queue: `gh pr edit <pr-number> --add-label "auto-merge"` (create the label first if missing: `gh label create auto-merge --color 0E8A16 --force`). On a repo with a merge queue, enqueue instead.
2. **Confirm the label is applied** — re-read the PR labels. If the apply failed, retry once; if it still fails, that is a hard blocker to escalate (do NOT fall back to waiting on CI yourself).
3. The Step 3b `awaiting-merge` milestone is already posted — that is the durable state.
4. Print the handoff line and STOP:

   ```
   Ship detached: PR #<pr-number> parked on auto-merge; run `full cycle issue <issue_number>` after merge to finish (resumes at ship-teardown), or let the fleet supervisor do it.
   ```

Do NOT wait for CI, do NOT merge, do NOT tear down, do NOT run report. **Leave the worktree in place** — the work is already pushed (WIP sync rule), and the tail run reuses or re-materializes it.

What completes the run later is unchanged machinery: gate-issue reads the `ship awaiting-merge` milestone and maps a merged PR to `resume_from=ship-teardown` (still-open → `ship-wait`), so the tail run re-enters at Step 6b/Step 7 and finishes with the final milestone and the report.

Steps 4 through 8 below are the **attached** path (and the tail run's path on resume).

### Step 4: Wait for CI — stable-confirmation gate (stay in this turn)

**You MUST stay in THIS turn until the gate passes — do NOT background the wait.** No
`Monitor`, no `run_in_background` poll, no "I'll be notified when CI finishes," no ending
your turn while checks are still pending. CI here takes ~18–20 minutes (the backend `Tests`
integration job runs on Blacksmith); you wait by re-running
a short foreground poll **batch** yourself, back-to-back, until it reports green or failing.
A backgrounded or deferred wait is the #1 cause of a PR that goes green but **never merges**
because the turn ended before Step 6 — do not do it. (Why a batch and not one long loop: the
Bash tool caps a single call at a few minutes and blocks open-ended foreground `sleep`, so a
single ~20-minute poll loop cannot complete in one call — it gets killed mid-wait. The fix is
to make "keep waiting" an explicit **same-turn re-run** of a short bounded batch.)

**Do NOT merge until checks are CONFIRMED green, and guard against false positives.**
The GitHub check-runs API (and `gh pr checks --watch`) intermittently report a phantom
`completed`/`success` for a job that is still running, and can momentarily show all-pass
*before* a slow required job (e.g. the backend integration suite) has even registered.
Merging on a single transient read ships an unverified build. This has bitten real runs —
treat one green read as **unconfirmed**.

The merge gate has two conditions, **both confirmed on two consecutive reads ≥20s apart**:

1. `mergeStateStatus` is `CLEAN` (via `gh pr view <pr-number> --json mergeStateStatus`).
2. `gh pr checks <pr-number>` shows **zero** checks `pending` and **zero** `fail`/`cancel`.

Run this **bounded poll batch**. It does up to 6 reads (~2.5 min) then EXITS with a
`RESULT=` line — short enough to finish inside the Bash tool timeout. It persists the
consecutive-clean counter to a file so two-in-a-row survives across batch re-runs, and reads
`bucket` from `--json` (machine-readable; `skipping` is correctly ignored, so a skipped job
never blocks and an unreported-checks window never false-greens):

```bash
SF=/tmp/ship-stable-<pr-number>; stable=$(cat "$SF" 2>/dev/null || echo 0); i=0
while [ "$i" -lt 6 ]; do
  i=$((i + 1))
  mss=$(gh pr view <pr-number> --json mergeStateStatus --jq '.mergeStateStatus' 2>/dev/null)
  checks=$(gh pr checks <pr-number> --json bucket 2>/dev/null)
  if [ -z "$checks" ]; then pend=1; fail=0; else      # no checks reported yet => not ready
    pend=$(printf '%s' "$checks" | jq '[.[]|select(.bucket=="pending")]|length')
    fail=$(printf '%s' "$checks" | jq '[.[]|select(.bucket=="fail" or .bucket=="cancel")]|length')
  fi
  echo "read $i/6: mss=$mss pending=$pend failing=$fail stable=$stable"
  if [ "$fail" -gt 0 ]; then echo 0 >"$SF"; echo "RESULT=failing"; exit 0; fi
  if [ "$mss" = "CLEAN" ] && [ "$pend" -eq 0 ]; then stable=$((stable + 1)); else stable=0; fi
  if [ "$stable" -ge 2 ]; then echo 0 >"$SF"; echo "RESULT=green"; exit 0; fi
  [ "$i" -lt 6 ] && sleep 25
done
echo "$stable" >"$SF"; echo "RESULT=pending stable=$stable"
```

When you run it, set the Bash tool `timeout` to `200000` (200s) so the batch isn't cut off
mid-poll. Then act on the `RESULT` line:

- `RESULT=green` ⇒ two consecutive confirmed-clean reads. Go to **Step 6 now, in this same
  turn**.
- `RESULT=failing` ⇒ go to **Step 5**.
- `RESULT=pending` ⇒ **immediately run the batch again. Do NOT yield, do NOT background, do
  NOT end your turn.** Keep re-running back-to-back until it returns green or failing (~8–10
  batches covers a ~20-min CI; do NOT stop polling at the 12-min mark). Each re-run resumes
  the counter from the file.
- `mergeStateStatus` values other than `CLEAN` (`UNSTABLE`, `BLOCKED`, `BEHIND`, `UNKNOWN`)
  reset the counter inside the batch — `UNSTABLE`/`UNKNOWN` usually just means not every check
  has reported yet; keep re-running, do not treat it as a failure on its own.

**Known transient false-blocks (re-run once, don't escalate):** if a required check fails
but the signature is a known-flaky/environmental one *unrelated to the diff* — a codegen
race (e.g. a docs/`.source` "not a module" / missing-property typecheck on a PR that
touches no docs), or a freshly-published dependency advisory the PR never introduced
(OSV-Scanner on a transitive dep) — re-run that job once (`gh run rerun <run-id> --failed`)
before treating it as a real Step-6 failure. If a security advisory is genuinely on `main`
too, the fix is a separate dependency-bump PR, not this one.

### Step 5: Handle CI Failures (max 2 attempts)

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
6. Wait for CI again (go back to Step 4)
7. If CI fails after 2 fix attempts: do not merge a red build. Stop and hand off —
   apply the `decision-pending` label (`gh label create decision-pending --color 5319E7
   --description "Blocked on a human decision" --force`), remove `in-progress`, and
   comment on the ORIGINAL issue with the failing job, the root cause you found, what
   you tried, and why it's still red. Do not open a new issue. End the run here.

### Step 6: Merge

**Only after the Step 4 gate exited with `stable=2`.** Re-confirm with one final read
immediately before merging — `gh pr view <pr-number> --json mergeStateStatus` must still
be `CLEAN`. If it regressed to `UNSTABLE`/`BLOCKED` (a check re-queued or a new push
landed), return to Step 4; never merge on a stale green.

All checks confirmed green — merge:

```bash
gh pr merge <pr-number> --squash
```

Do NOT use `--delete-branch` — it fails from worktrees. Branch cleanup happens in Step 7.

Clean up issue labels:

```bash
gh issue edit <issue_number> --remove-label "in-progress"
```

### Step 6b: Confirm the merge before doing ANYTHING else

**This is a hard gate — do not skip it.** Immediately after Step 6, run:

```bash
gh pr view <pr-number> --json mergedAt,state
```

`mergedAt` MUST be non-null **and** `state` MUST be `MERGED`. If it is not, **you are not
done** — the merge did not happen; return to Step 4 / Step 6 and drive it to a real merge.
Never emit an idle notification, end your turn, or proceed to Teardown (Step 7) or Report
(Phase 6) with an unmerged PR. "PR opened and checks passing" is **not** a completed run.
A background agent that idles green-but-unmerged here is the single most common failure of
this workflow — this gate exists to stop it.

A confirmed merge is **necessary but NOT sufficient**: it puts code on the default branch,
not in front of users. Continue to Step 6c — do not treat this gate as the finish line.

### Step 6c: Confirm the merge REACHED PRODUCTION

**A merge is not a release.** Step 6b proves the code is on the default branch; it does
NOT prove a single user can see it. Treat "merged" as done and you will report success on
code that is live to nobody.

This is not hypothetical. On a repo whose merges are performed by a bot token, GitHub
deliberately does NOT fire `on: push` workflows for those pushes — so the deploy pipeline
never runs, and the merged code sits on the default branch until some *unrelated* human
push happens to carry it out. Observed 2026-07-17 (imboard #2714): four PRs merged green,
`MERGE_COMMIT` captured, zero deploys; they reached production hours later bundled into a
stranger's push. Every signal this workflow checks said "shipped".

**Skip this step and the workflow's success token is a lie.**

1. **Find the project's deploy mechanism.** Do not assume it exists or that it is
   automatic. Look for a deploy workflow (`gh workflow list`), a `deploy`/`release`
   script, or a documented runbook in the repo guide. If the project has NO deploy step
   (a library, a docs site auto-built on push, an app deployed by an external system you
   cannot observe), record `DEPLOYED=N/A — <reason>` and move on. That is a legitimate
   outcome; silence is not.

2. **Check whether a deploy already carries your merge.** A successful deploy run whose
   commit CONTAINS `MERGE_COMMIT` (it need not equal it — a later deploy carrying your
   commit still ships it):

   ```bash
   # newest successful deploy runs, with the SHA each one shipped
   gh run list --workflow <deploy-workflow> --limit 5 \
     --json headSha,status,conclusion,createdAt \
     --jq '.[] | select(.conclusion=="success") | .createdAt + " " + .headSha'
   # is your merge contained in what shipped?
   git fetch origin --quiet
   git merge-base --is-ancestor <MERGE_COMMIT> <deployed_sha> && echo SHIPPED || echo NOT-SHIPPED
   ```

3. **If nothing ships it within ~5 minutes, dispatch the deploy yourself**, then confirm
   it succeeded:

   ```bash
   gh workflow run <deploy-workflow> [-f environment=production]   # inputs vary — read the workflow
   ```

   Deploying deliberately is SAFER than the default. The merged code is going to
   production regardless — the only question is whether it goes at a moment nobody chose,
   unattended, bundled with an unrelated change, and attributed to whoever pushed next.

4. **A failed deploy is a hard blocker.** Escalate on the issue (comment + remove
   `in-progress`) with the run URL. Do NOT report the run as complete. Do NOT retry blindly
   more than once — a red deploy on the default branch may be affecting live users and is a
   human's call.

5. **Record `DEPLOYED`** — the shipped SHA + the run URL — and pass it to Phase 6.

**Only now is the work shipped.**

### Step 7: Teardown

**Step 7.0 — release per-worktree test resources first.** If the repo has `scripts/ensure-test-env.sh` (or `main/scripts/ensure-test-env.sh`), run it with `--teardown` from the worktree BEFORE removing the worktree:

```bash
bash scripts/ensure-test-env.sh --teardown
```

This drops the worktree's isolated test database and S3 prefix. Skipping it leaks one database per run; shared-tier Atlas clusters cap at 500 collections cluster-wide and every leaked `imboard_test_<slug>` DB eats ~40 of them — once full, every integration test fails with `cannot create a new collection -- already using 500 collections of 500`. Record `test_env=torn-down|none` in the final ship milestone.

**Verify cleanup before claiming it.** `cleanup=pool_returned` may only be posted after confirming it: `npx -y @ai-dossier/worktree-pool@^0.5.1 status` no longer lists the entry as assigned AND `git worktree list` no longer contains the path. If the return errored or the state is inconsistent, post `cleanup=failed-<step>` instead — a milestone claiming completion is not proof of completion (imboard#3692, ai-dossier#453).

**Prerequisite: Step 6b (merge confirmed) AND Step 6c (deploy confirmed or `N/A`) must be
complete.** Do not tear down before the
merge is confirmed.

1. `cd` back to `original_dir` (if provided)
2. **Try to return the worktree to the pool** (if `pool_claimed` is true):
   ```bash
   npx -y @ai-dossier/worktree-pool@^0.5.1 return --path <worktree_path> 2>/dev/null
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
   Deleting the remote branch also deletes the WIP history accumulated during this run
   (plan/implement/review's `wip(...)` commits) — that is fine and intended; the squash-merge
   commit already landed on the base branch as the durable artifact.

### Step 8: Runstate Milestone (final)

Post the second and final ship milestone, after merge and teardown. (Not reached in `ship_mode=detached` — the tail run posts it.) This is the last step of the phase — if ship aborts (CI red after 2 attempts, merge conflict, failed deploy), post `--status blocked --kv reason=<short-slug>` instead and stop. Do not skip this in nested or fleet mode — it is the only state that survives the session.

```bash
ai-dossier runstate post --issue <issue_number> --phase ship --status done --run <run_id> \
  --kv pr=<pr-number> \
  --kv merge_commit=<short sha from Step 6b> \
  --kv ci_fix_attempts=<n> \
  --kv cleanup=pool_returned|worktree_removed|skipped \
  --kv test_env=torn-down|none
```

The CLI stamps `at=` and computes `next=report` — do not pass either. It validates phase, status, and keys and refuses a malformed milestone; never hand-write the comment instead. On an aborted phase: `--status blocked --kv reason=<short-slug>`. `ci_fix_attempts` is how many Step 5 fix-and-push cycles ran (0 if CI was green first time). Values contain no spaces (use `-` or `,`); paths are absolute.

## Output

- `pr_number`: the created PR number
- `pr_url`: the PR URL
- `ship_mode`: attached | detached — which path ran
- `merge_status`: merged | failed | parked (detached: PR open on auto-merge, no merge attempted)
- `target_branch`: the branch merged into
- `cleanup`: pool_returned | worktree_removed | skipped
- `ci_fix_attempts`: number of CI fix-and-push cycles run in Step 5
- Posts TWO runstate milestones to the issue (`phase=ship`: `awaiting-merge` before the CI wait, then `done`) — in `detached` mode only the first, and the tail run posts the second

## Validation

- [ ] Changes committed with conventional commits format
- [ ] Branch pushed to remote
- [ ] PR created targeting correct base_branch
- [ ] PR body includes the Acceptance Criteria section from `ac_results` (when non-empty)
- [ ] `ship_mode` was honored: `detached` stopped after the label + `awaiting-merge` milestone with the handoff line printed (no CI wait, no merge, no teardown, no report); `attached` ran through to the final milestone
- [ ] Detached only: the `auto-merge` label was applied and confirmed present, and the worktree was left in place
- [ ] CI passed (or failures fixed within 2 attempts)
- [ ] CI confirmed green on two consecutive stable polls — not a single transient success
- [ ] CI wait done in-turn (foreground batch re-runs) — never backgrounded or deferred
- [ ] PR merged (squash)
- [ ] Merge confirmed: `gh pr view` shows `mergedAt` non-null and `state` `MERGED` (Step 6b)
- [ ] Deploy confirmed: a successful deploy run CONTAINS `MERGE_COMMIT`, or `DEPLOYED=N/A` with a reason (Step 6c) — merged is not shipped
- [ ] in-progress label removed
- [ ] Worktree returned to pool or removed
- [ ] Returned to original directory
- [ ] `scripts/ci-parity.sh` was run before committing when present
- [ ] Runstate milestones were posted via `ai-dossier runstate post` (`awaiting-merge` before the CI wait — with `--next ship` — and, on the attached/tail path, the final one after teardown)

## Troubleshooting

**CI fails after fixes**: See Step 5 item 7 — after 2 attempts, stop and hand off on
the issue (`decision-pending` label + comment). Do not open a new issue. May be an
infrastructure issue rather than a code issue — say so in the comment.

**Phantom success / flaky check status**: the checks API can report a transient
`success` while a required job is still running, or all-pass before a slow job registers.
Never merge on one read — require two consecutive `CLEAN` + zero-pending reads (Step 4).

**Merge stall / "I'll be notified when CI is done"**: the most common failure of this phase
is the agent backgrounding the CI wait (a `Monitor`, a `run_in_background` poll, or just
ending the turn to "wait for notification") — the PR then goes green but never merges. Never
do that. Step 4 is a foreground, same-turn loop: run the bounded batch, and on `RESULT=pending`
run it again immediately. Stay in the turn until `RESULT=green` (→ merge) or `RESULT=failing`.

**Merge conflicts**: Needs human judgment. Stop and hand off on the issue
(`decision-pending` label + comment describing the conflicting files and why an
automatic resolution isn't safe) — do not guess at a resolution, do not open a new issue.

**Detached run looks unfinished**: it is — by design. A `ship awaiting-merge` milestone with no `ship done` after it is a parked PR, not a failure. The tail run (`full cycle issue <n>`) resumes at `ship-teardown` once the PR merges.

**`--delete-branch` fails in worktree**: Expected — don't use it. Clean up in Step 7.

**Pool return fails**: Not an error — fall back to manual worktree remove.
