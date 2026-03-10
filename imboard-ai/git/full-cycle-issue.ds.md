---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "full-cycle-issue",
  "title": "Full Cycle Issue Workflow",
  "version": "2.6.0",
  "status": "Draft",
  "last_updated": "2026-03-10",
  "objective": "Take a GitHub issue from start to merged PR autonomously — setup, implement, test, commit, push, PR, parallel review, and merge with zero unnecessary interruptions",
  "category": [
    "development"
  ],
  "tags": [
    "github",
    "issues",
    "workflow",
    "autonomous",
    "full-cycle",
    "pr",
    "merge"
  ],
  "risk_level": "medium",
  "requires_approval": false,
  "risk_factors": [
    "modifies_files",
    "network_access",
    "creates_pull_request",
    "merges_code"
  ],
  "destructive_operations": [
    "Creates new git branch",
    "Creates new git worktree",
    "Pushes branch to remote",
    "Creates and merges pull request",
    "Deletes branch after merge"
  ],
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "d6d11d542b3d42c01b96b30fd781e193945e84fee363a7e92cab12f461798925"
  }
}
---

# Full Cycle Issue Workflow

## Objective

Take a GitHub issue from start to merged PR autonomously. For small-to-medium issues where requirements are clear.

## Guiding Principle

**Do not stop to ask yes/no questions.** Only pause when:
- The issue description is too vague to implement
- A business/product/design decision is genuinely ambiguous
- Tests fail after 2 fix attempts with unclear path forward
- Merge conflicts require human judgment

Do NOT ask about: file names, branch names, commit messages, PR descriptions, whether to proceed, or any mechanical decision.

## Prerequisites

- [ ] Git is installed and configured
- [ ] GitHub CLI (gh) is installed and authenticated
- [ ] You are in a git repository with GitHub as a remote
- [ ] You have push access

## Actions to Perform

### Phase 0: Self-Check (Quick Gate ~30s)

Lightweight safety gate before committing to the full workflow. No codebase exploration — just check issue metadata.

1. Extract the issue number from user input
2. Fetch issue metadata:
   ```bash
   gh issue view <number> --json state,labels,body
   ```
3. **Hard blocks** — if ANY of these are true, abort immediately with a comment on the issue:
   - **Issue is closed**: `state == "CLOSED"`
   - **Has label `decomposed`**: Issue was already decomposed into sub-issues by triage
   - **Has label `needs-clarification`**: Issue was triaged as not ready
   - **Has label `epic`**: Issue is an epic, not directly implementable
   - **Has open dependency**: Body contains `Depends on #N` (case-insensitive) where issue #N is still open:
     ```bash
     # For each "Depends on #N" found in the body:
     gh issue view <N> --json state --jq '.state'
     ```
     If the referenced issue state is `OPEN`, it's a hard block.

   **On hard block**: Post a comment and stop:
   ```bash
   gh issue comment <number> --body "**Full-cycle aborted**: <reason>. Resolve the blocker and re-run."
   ```
   Do NOT proceed to Phase 1.

4. **Soft warnings** — count how many of these are true:
   - Body is empty or less than 50 characters
   - Body contains research keywords: `evaluate`, `research`, `explore`, `investigate`, `compare` (case-insensitive)
   - Issue has no labels at all

   If **2 or more** soft warnings: log a warning line but continue:
   ```
   ⚠ Phase 0: 2 soft warnings (short body, no labels) — proceeding with caution
   ```
   If 0-1 soft warnings: proceed silently.

### Phase 1: Setup

1. Extract the issue number from user input (already done in Phase 0)
2. **Pre-flight: clean stale worktrees.** Previous runs may have left zombie worktrees that lock branches (especially `main`). This causes "branch is checked out in another worktree" errors.
   ```bash
   git worktree list
   ```
   For each worktree listed:
   - If it checks out `main` but is NOT the repo root → it's stale. Remove it:
     ```bash
     git worktree remove <path> --force
     ```
   - If its branch was already merged and deleted on remote → it's stale. Remove it:
     ```bash
     git worktree remove <path>
     ```
   - Run `git worktree prune` to clean up any broken references
   After cleanup, verify `main` is not locked:
   ```bash
   git worktree list | grep -w main
   ```
   Only the repo root (or its bare checkout) should show `main`.
3. **Claim the issue** — make it visible that work is in progress:
   ```bash
   # Ensure the label exists
   gh label create "in-progress" --color "FBCA04" --description "Actively being worked on" --force
   # Assign to current user and label
   gh issue edit <number> --add-label "in-progress" --add-assignee "@me"
   # Post agent context comment
   gh issue comment <number> --body "$(cat <<'EOF'
   **Agent pickup** — work started.
   - **Agent**: Claude (full-cycle-issue workflow v2.6.0)
   - **Branch**: (will update after setup)
   - **Started**: $(date -u +%Y-%m-%dT%H:%M:%SZ)
   EOF
   )"
   ```
4. **Record the original working directory** — you will return here after merge
5. Run the setup workflow:
   ```bash
   ai-dossier run imboard-ai/git/setup-issue-workflow
   ```
6. Provide the issue number when prompted
7. **When asked where to work, always choose option 1 (create a new git worktree)**. Do not use current directory or custom path — full-cycle must be isolated. The setup workflow will automatically try the worktree pool first for instant setup; if no pool is available it falls back to cold worktree creation.
8. Note the worktree path and branch name from the setup output
9. **Record whether the worktree was claimed from the pool** — check the setup output for "Claimed pre-warmed worktree from pool". You will need this in Phase 9 to decide whether to return the worktree to the pool.
10. `cd` into the worktree directory — **all subsequent work happens here**
11. **Verify you are in a worktree.** This is a hard gate — do NOT proceed without passing it.
    ```bash
    pwd | grep -q "worktree" && echo "OK: in worktree" || echo "FAIL: not in worktree"
    ```
    If `FAIL`: **STOP IMMEDIATELY.** Do not implement, commit, or push anything. Instead:
    - Report exactly what happened: what directory you're in, what `git worktree list` shows, what the setup workflow output was
    - Comment on the issue: `gh issue comment <number> --body "Agent aborted: not in a worktree. pwd=$(pwd). Needs investigation."`
    - Remove the `in-progress` label: `gh issue edit <number> --remove-label "in-progress"`
    - Exit. Do not attempt to work in the current directory as a fallback.
12. Update the issue comment with the branch name:
    ```bash
    gh issue comment <number> --body "Branch: \`<branch-name>\` | Worktree: \`<worktree-path>\`"
    ```

### Phase 2: Understand & Plan

1. Read issue details: `gh issue view <number> --json title,labels,body,assignees`
2. Read PLANNING.md created by setup
3. Explore relevant code
4. If clear enough, proceed immediately
5. If genuinely ambiguous, ask ONE focused question then proceed

### Phase 3: Implement

1. Implement the solution following existing code patterns
2. Keep changes minimal and focused
3. Do not add tests inline — Phase 4 handles testing separately
4. **Before building**, run the project's auto-fixer to avoid lint iteration loops:
   - Node.js with biome: `npx biome check --write .`
   - Node.js with eslint: `npx eslint --fix .`
   - Python with ruff: `ruff check --fix .`
   - Or whatever the project's `lint:fix` script is (check package.json / Makefile)

### Phase 4: Test

1. **Detect test framework**: Look for jest.config, vitest.config, .mocharc, pytest.ini, or test scripts in package.json / pyproject.toml
2. **If tests exist for changed files** — run them. Fix failures (max 2 attempts). If still failing with unclear path, ask the user.
3. **If no tests exist for the changed code** — create focused unit tests:
   - Test the public API of changed/new modules
   - Cover happy path + key edge cases + error paths
   - Follow existing test patterns and conventions in the repo
   - Place tests where the project convention expects them (e.g., `__tests__/`, `*.test.ts`, `*.spec.ts`)
4. **Run the full test suite** to catch regressions (e.g., `npm test`)
5. **If tests fail**, check whether they are **pre-existing failures** by running the same tests on the base branch:
   ```bash
   git stash && git checkout main && npm test 2>&1 | tail -5 && git checkout - && git stash pop
   ```
   If the same tests fail on main, they are pre-existing — ignore them and proceed. Only fix failures caused by your changes (max 2 attempts).

### Phase 5: Review & Fix

**CRITICAL: You must still be `cd`'d into the worktree directory for this phase.** The review agents need to read files and apply fixes. Do NOT proceed to Phase 6 (Commit) until this phase completes.

Run `pwd` to confirm you are in the worktree. If not, `cd` back into it.

Get the changed files list: `git diff --name-only` (unstaged changes — we haven't committed yet)

Run 5 focused review agents **in parallel** using the Agent tool. Launch all 5 simultaneously:

#### Classification Criteria (applies to ALL review agents)

Every review agent must classify each finding as follows:

- **Fix now** (default): Fix it yourself. Bugs, wrong text, missing validation, bad names,
  missing error handling, code duplication, doc inaccuracies, type improvements, refactoring
  — fix them all. If you can write the code, it's "Fix now". No exceptions for severity or
  scope — minor and major findings alike get fixed in-place.
- **Escalate**: ONLY for findings where ALL three of these are true:
  (a) The fix would change user-facing behavior or public API semantics
  (b) You cannot fully verify the fix with existing tests
  (c) It requires a product/business decision (e.g., "should we deprecate this?",
      "is this a breaking change we accept?")
  If any of (a), (b), (c) is false, it's "Fix now", not "Escalate".

**Never escalate**: code quality, documentation gaps, refactoring suggestions, type
improvements, minor bugs, "consider doing X" opinions. Fix them or skip them.

#### Agent 1: DRY Review

> You are reviewing uncommitted changes for DRY (Don't Repeat Yourself) violations.
> AI agents frequently rewrite code that already exists in the codebase — your job is to catch this.
>
> For each changed file:
> 1. Read the file fully
> 2. Search the **entire codebase** for existing functions, utilities, or patterns that do the same thing
> 3. Flag duplicated logic (>5 similar lines), reimplemented helpers, or missed utility reuse
> 4. Check if anything was reimplemented that's available in project dependencies
>
> Classify findings per the Classification Criteria above.
> If none found, report "No DRY violations found."

#### Agent 2: Security Review

> You are reviewing uncommitted changes for security vulnerabilities.
>
> Check every changed file for:
> - Injection (SQL, command, template, path traversal)
> - XSS (unescaped user input in HTML/JSX/templates)
> - Auth/authz gaps
> - Hardcoded secrets, API keys, tokens
> - Insecure patterns, unsafe deserialization
> - Missing input validation at system boundaries
> - OWASP Top 10
>
> Classify findings per the Classification Criteria above.
> If none found, report "No security issues found."

#### Agent 3: Supportability Review

> You are reviewing uncommitted changes for supportability — can someone debug and operate this code in production?
>
> Check every changed file for:
> - Error messages: Are they actionable? Do they include context (what failed, what was expected)?
> - Logging: Are key operations logged? Can you trace a request through the system?
> - Error handling: Are errors caught with useful context, or do they bubble as cryptic stack traces?
> - Failure modes: What happens when external calls fail? Is there graceful degradation?
>
> Classify findings per the Classification Criteria above.
> If none found, report "No supportability issues found."

#### Agent 4: Maintainability Review

> You are reviewing uncommitted changes for maintainability — will the next developer understand and safely modify this code?
>
> Check every changed file for:
> - Unclear or misleading names
> - Functions >50 lines or deeply nested (>3 levels)
> - Magic numbers/strings without named constants
> - Tight coupling that blocks testing or reuse
> - Missing TypeScript types (any, implicit any)
> - Dead code, unused imports, unreachable branches
> - Leftover console.log / debugger statements
> - TODO/FIXME/HACK without issue references
>
> Classify findings per the Classification Criteria above.
> If none found, report "No maintainability issues found."

#### Agent 5: Documentation Review

> You are reviewing uncommitted changes for documentation gaps and inaccuracies.
>
> Check:
> 1. **README**: Does it still accurately describe the project? Are new features/commands/options documented?
> 2. **Doc files** (docs/, *.md): Are any now outdated or incorrect because of the code changes?
> 3. **Code comments**: Are existing comments still accurate? Are complex new sections missing explanations?
> 4. **API surface**: If public APIs changed, are type definitions / JSDoc / OpenAPI specs updated?
> 5. **Examples**: Do code examples in docs still work?
>
> Classify findings per the Classification Criteria above.
> If none found, report "No documentation issues found."

#### After All Agents Complete

**Store the review results** — you will need them for the final report in Phase 10. Track:
- `review_fixed`: list of findings that were fixed in-place (file, line, what was fixed)
- `review_escalated`: list of findings that need human judgment (for GH issue creation after PR)
- `review_clean`: list of categories with zero findings

1. **Collect** all findings from the 5 agents
2. **Fix ALL "Fix now" findings** — use the Edit tool directly. These can be non-trivial: refactors, adding error handling, fixing historic lint issues in touched files, etc.
3. **Re-run tests** after fixes to ensure nothing broke. If a fix breaks tests, revert that specific fix and reclassify as Escalate.
4. **Run lint auto-fixer** to clean up formatting:
   - Node.js with biome: `npx biome check --write .`
   - Node.js with eslint: `npx eslint --fix .`
   - Python with ruff: `ruff check --fix .`
   - Or whatever the project's `lint:fix` script is

### Phase 6: Commit & Push

Implementation and review fixes go together in one clean commit.

1. Stage relevant files (never `.env`, credentials, secrets)
2. Commit with conventional commits format:
   ```
   feat|fix|chore: <description>

   Closes #<issue-number>

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```
3. Push: `git push -u origin <branch-name>`

### Phase 7: Create PR

```bash
gh pr create --title "<short title>" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points>

Closes #<issue-number>

## Test plan
- <how to verify>

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

#### Create GH Issues for Escalated Findings

After the PR is created, handle any Escalate findings from Phase 5.

**Consolidation rule**: Group all escalated findings from the same review category
(DRY, Security, Supportability, Maintainability, Documentation) into a **single GH issue
per category**. This means at most 5 issues total (one per review agent), but typically 0.

Most PRs should have **zero** escalated issues. If you're escalating more than 2 total,
you are almost certainly being too aggressive — re-evaluate each finding against the
three-part test (user-facing behavior change + can't verify with tests + needs product decision).

- **For each category with escalated findings**: Create one consolidated GH issue:
  ```bash
  gh issue create --title "<Category>: escalated findings from PR #<pr-number>" --label "review" --body "$(cat <<'EOF'
  **Source**: PR #<pr-number>, found during <Category> review of <branch>

  ### Findings

  1. **<file>:<lines>** — <description>
     **Why escalated**: <why this needs human judgment>

  2. **<file>:<lines>** — <description>
     **Why escalated**: <why this needs human judgment>
  EOF
  )"
  ```
- If there are zero Escalate findings (the common case), create no GH issues at all.

### Phase 8: Wait for CI & Merge

**Prerequisite: Phase 7 (Create PR) must be complete** — PR created, escalated issues filed.

1. **Wait for CI to complete**. Do NOT merge until all checks pass.
   ```bash
   gh pr checks <pr-number> --watch --fail-fast
   ```
   This blocks until all checks finish. If the command is not available or times out, poll manually:
   ```bash
   gh pr checks <pr-number>
   ```
   Repeat every 30 seconds until all checks show `pass` or `fail`.

2. **If CI fails** — diagnose and fix (max 2 attempts):
   a. Identify which job failed:
      ```bash
      gh pr checks <pr-number>
      ```
   b. Get the failed run ID and view its logs:
      ```bash
      gh run view <run-id> --log-failed
      ```
   c. Read the failure output, identify the root cause, fix the code
   d. Run the failing command locally first (e.g., `npm run lint`, `npm test`) to confirm the fix
   e. Commit and push:
      ```bash
      git add <files> && git commit -m "fix: CI failure — <what was wrong>

      Co-Authored-By: Claude <noreply@anthropic.com>"
      git push
      ```
   f. Wait for CI again (go back to step 1)
   g. If CI fails after 2 fix attempts, ask the user — do not merge a red build

3. **All checks green** — merge:
   ```bash
   gh pr merge <pr-number> --squash
   ```
   Do NOT use `--delete-branch` here — it fails when merging from a worktree because it tries to checkout the base branch locally. The remote branch is cleaned up by GitHub when the PR is merged with "delete branch on merge" repo setting, and the local branch is cleaned up in Phase 9.

4. Clean up issue labels (the `Closes #<number>` in the PR body auto-closes the issue; remove the in-progress label):
   ```bash
   gh issue edit <number> --remove-label "in-progress"
   ```

### Phase 9: Teardown

**Prerequisite: Phase 8 (Merge) must be complete.** Do not tear down the worktree before the PR is merged.

1. `cd` back to the **original working directory** (recorded in Phase 1)
2. **Try to return the worktree to the pool** (if the worktree was claimed from the pool in Phase 1, Step 9):
   ```bash
   npx worktree-pool return --path <worktree-path> 2>/dev/null
   ```
   - If the command succeeds: the worktree is recycled back to the pool for reuse. Skip steps 3-5.
   - If the command fails (pool not installed, worktree not from pool, or return error): continue with manual cleanup below.
3. Remove the worktree:
   ```bash
   git worktree remove <worktree-path>
   ```
4. If the local branch still exists, delete it:
   ```bash
   git branch -d <branch-name> 2>/dev/null || git branch -D <branch-name>
   ```
5. Clean up the remote branch if it still exists:
   ```bash
   git push origin --delete <branch-name> 2>/dev/null || true
   ```

### Phase 10: Report

Print a single consolidated report. This is the **only** summary the user sees — include everything.

```
## Full Cycle Complete

**Issue**: #<number> — <title>
**PR**: #<pr-number> (merged, squashed)
**Branch**: <branch-name>

### Changes
<1-3 sentence summary of what was implemented>

### Tests
- <created N new test files / ran N existing test files>
- All passing

### Review Results

**Fixed in this PR** (<N> review fixes included in commit):
- <file>:<line> — <what was fixed>
- <file>:<line> — <what was fixed>

**Escalated** (<N> issues created, consolidated per review category — needed human judgment):
- #<issue> — <title> (or "None")

**Clean categories** (no findings):
- <list categories with zero findings>

### Cleanup
Worktree removed. Back in original directory.
```

If pool return was used instead of worktree removal:
```
### Cleanup
Worktree returned to pool for reuse. Back in original directory.
```

If there were no review findings at all, replace the Review Results section with:
```
### Review Results
All 5 reviews passed clean — no findings.
```

## Validation

- [ ] Phase 0 self-check passed (no hard blocks)
- [ ] Issue fetched and understood
- [ ] Branch and worktree created (via pool claim or cold creation)
- [ ] Implementation addresses requirements
- [ ] Tests exist and all pass (created if missing)
- [ ] 5 parallel reviews completed (before commit)
- [ ] Review fixes applied in-place
- [ ] Tests re-run after review fixes
- [ ] Committed with conventional commit message (implementation + review fixes together)
- [ ] PR created linking to issue
- [ ] Escalated findings consolidated per review category (typically 0 issues)
- [ ] PR merged
- [ ] Worktree returned to pool or removed
- [ ] Returned to original working directory

## Troubleshooting

**`gh` not found**: Install GitHub CLI: https://cli.github.com/
**CI fails after fixes**: Ask user — may be infrastructure issue
**Merge conflicts**: Ask user — needs human judgment
**Vague issue**: Ask ONE clarifying question, then proceed
**No test framework detected**: Default to vitest (Node.js) or pytest (Python); install if needed
**`--delete-branch` fails in worktree**: This is expected — do not use `--delete-branch` with `gh pr merge` when working from a worktree. Clean up branches in Phase 9 instead.
**Pre-existing test failures**: Run tests on the base branch to confirm. Only fix failures caused by your changes.
**Review fix breaks tests**: Revert the specific fix and reclassify as Escalate. Do not let review fixes destabilize the build.
**Pool return fails**: Not an error — the worktree may not have been from the pool. Fall back to manual `git worktree remove`.
