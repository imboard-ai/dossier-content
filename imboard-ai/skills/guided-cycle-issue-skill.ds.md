---dossier
{
  "dossier_schema_version": "1.1.0",
  "name": "guided-cycle-issue-skill",
  "title": "Guided Cycle Issue",
  "version": "1.0.0",
  "status": "Draft",
  "objective": "Collaborative issue workflow: plan with user review, implement, optional visual review for FE changes, then autonomous ship and rich report",
  "description": "Collaborative issue workflow with plan review and optional visual review. Use when user says 'guided cycle', 'plan issue', 'lets discuss issue', 'work on issue with review', 'guided issue'",
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "category": [
    "skills"
  ],
  "tags": [
    "github",
    "workflow",
    "collaborative",
    "skill",
    "issue"
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "PLACEHOLDER"
  }
}
---

# Guided Cycle Issue

Collaborative issue workflow with human checkpoints. Unlike full-cycle (fire-and-forget), this workflow pauses for plan review and optional visual review before shipping.

## Project Parameters

- **warmup_dossier**: `imboard-ai/imboard/warm-worktree`

## Flags

Parse these from the user's request:
- `--review`: Force visual review checkpoint after implementation (regardless of file types)
- `--no-review`: Skip visual review checkpoint (regardless of file types)
- `--base <branch>`: Override the target branch (bypasses issue body parsing)

## Workflow

### Phase A: Gate
1. Extract the issue number from the user's request
2. Run: `ai-dossier run imboard-ai/git/gate-issue`
3. Provide the issue number when prompted
4. If the gate fails (hard block), stop and report the reason to the user
5. Note the `base_branch` from the gate output

If user provided `--base <branch>`, use that instead of the gate's extracted base_branch.

### Phase B: Setup
1. Run: `ai-dossier run imboard-ai/git/setup-issue-workflow`
2. Pass through:
   - The `warmup_dossier` parameter above
   - The resolved `base_branch`
3. When asked where to work, always choose option 1 (create a new git worktree)
4. Note the worktree path, branch name, and whether the worktree was claimed from the pool
5. `cd` into the worktree directory
6. Record the original working directory — you will return here after teardown

### Phase C: Plan + Discuss (CHECKPOINT)
1. Run: `ai-dossier run imboard-ai/git/plan-issue`
2. Pass through the issue number, base_branch, and worktree path
3. **CHECKPOINT — Present the plan to the user:**
   - Show the full contents of the generated `PLANNING-{number}-{slug}.md`
   - Ask: "Does this plan look right? Any changes before we implement?"
   - Iterate with the user until they approve the plan
   - Update the planning file with any changes discussed
4. Do NOT proceed to Phase D until the user approves the plan

### Phase D: Implement
1. Run: `ai-dossier run imboard-ai/git/implement-issue`
2. Pass through the planning file path and base_branch
3. After implementation completes, determine if visual review is needed:

**Visual review decision:**
- If `--review` flag was set: visual review is required
- If `--no-review` flag was set: skip visual review
- Otherwise, auto-detect: check if frontend files were changed:
  ```bash
  git diff --name-only | grep -E '\.(tsx|jsx|css|scss|vue|svelte)$'
  ```
  If any match → visual review required.

4. **If visual review is required — CHECKPOINT:**
   - Tell the user: "Implementation complete. FE files were changed — let's review the UI."
   - Describe what was built and what the user should see
   - If screenshot MCP tools are available, take and show screenshots
   - Iterate with the user until they are satisfied with the visual result
   - Apply any requested changes
   - Do NOT proceed until the user approves

### Phase E: Review
1. Run: `ai-dossier run imboard-ai/git/review-issue`
2. This launches 5 parallel review agents internally
3. Collect the review results: `review_fixed`, `review_escalated`, `review_clean`

### Phase F: Ship
1. Run: `ai-dossier run imboard-ai/git/ship-issue`
2. Pass through: issue number, base_branch, review_escalated, worktree_path, original_dir, pool_claimed

### Phase G: Report
1. Run: `ai-dossier run imboard-ai/git/report-issue`
2. Pass through: issue number, pr_number, base_branch, review_fixed, review_escalated, review_clean, cleanup_method
