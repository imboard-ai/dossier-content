---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "full-cycle-issue",
  "title": "Full Cycle Issue Workflow",
  "version": "1.1.0",
  "status": "Draft",
  "last_updated": "2026-03-05",
  "objective": "Take a GitHub issue from start to merged PR autonomously — setup, implement, commit, push, PR, review, and merge with zero unnecessary interruptions",
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
    "hash": "08da6827578d352e5c1f37c660f11191b8a3b8c507983d5ae8217fcdd0534d58"
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

### Phase 1: Setup

1. Extract the issue number from user input
2. **Record the original working directory** — you will return here after merge
3. Run the setup workflow:
   ```bash
   ai-dossier run imboard-ai/development/git/setup-issue-workflow
   ```
4. Provide the issue number when prompted
5. Note the worktree path from the setup output
6. `cd` into the worktree directory — **all subsequent work happens here**

### Phase 2: Understand & Plan

1. Read issue details: `gh issue view <number> --json title,labels,body,assignees`
2. Read PLANNING.md created by setup
3. Explore relevant code
4. If clear enough, proceed immediately
5. If genuinely ambiguous, ask ONE focused question then proceed

### Phase 3: Implement

1. Implement the solution following existing code patterns
2. Keep changes minimal and focused
3. Run tests if available and fix failures
4. Do not add tests unless the issue specifically asks for them

### Phase 4: Commit & Push

1. Stage relevant files (never `.env`, credentials, secrets)
2. Commit with conventional commits format:
   ```
   feat|fix|chore: <description>

   Closes #<issue-number>

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```
3. Push: `git push -u origin <branch-name>`

### Phase 5: Create PR

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

### Phase 6: Review

1. Run `/review-branch` on the current branch
2. If fixes were made, push them: `git push`

### Phase 7: Merge

1. Check CI: `gh pr checks <pr-number> --watch`
   - No CI or all pass: proceed
   - Fails: fix (max 2 attempts), then ask user
2. Merge: `gh pr merge <pr-number> --squash --delete-branch`

### Phase 8: Teardown

1. `cd` back to the **original working directory** (recorded in Phase 1)
2. Remove the worktree:
   ```bash
   git worktree remove <worktree-path>
   ```
3. If the local branch still exists, delete it:
   ```bash
   git branch -d <branch-name>
   ```

### Phase 9: Report

```
Done. Issue #<number> merged via PR #<pr-number>.
Branch: <branch-name>
Changes: <1-2 sentence summary>
Worktree cleaned up.
```

## Validation

- [ ] Issue fetched and understood
- [ ] Branch and worktree created
- [ ] Implementation addresses requirements
- [ ] Tests pass (if they exist)
- [ ] Committed with conventional commit message
- [ ] PR created linking to issue
- [ ] Review run and fixes applied
- [ ] PR merged
- [ ] Worktree removed
- [ ] Returned to original working directory

## Troubleshooting

**`gh` not found**: Install GitHub CLI: https://cli.github.com/
**CI fails after fixes**: Ask user — may be infrastructure issue
**Merge conflicts**: Ask user — needs human judgment
**Vague issue**: Ask ONE clarifying question, then proceed
