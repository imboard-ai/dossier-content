---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "full-cycle-issue-skill",
  "title": "Full Cycle Issue",
  "version": "1.5.0",
  "status": "Draft",
  "objective": "Take a GitHub issue from start to merged PR autonomously",
  "description": "Full autopilot: gate, setup, plan, implement, test, review, commit, push, PR, merge, rich report. Use when user says 'full cycle issue', 'auto issue', 'autopilot issue', 'fire and forget'",
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
    "autonomous",
    "skill"
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "489e5aad3d9246e70909022de4ca7344c55ad71aef6f1e7495d44264314ef328"
  }
}
---

# Full Cycle Issue

## Project Parameters

- **warmup_dossier**: `imboard-ai/imboard/warm-worktree`

## Flags

Parse these from the user's request:
- `--base <branch>`: Override the target branch (bypasses issue body parsing)

## Steps

1. Extract the issue number from the user's request
2. Run: `ai-dossier run imboard-ai/git/full-cycle-issue --pull`
3. The workflow will use the `warmup_dossier` parameter above for worktree warmup (pnpm install + build shared-types, no .env copying)
4. If `--base` flag was provided, pass the base_branch parameter to the workflow
5. Follow ALL phases in the workflow output. Do not skip any.
6. Do not stop to ask yes/no questions. Only pause if the issue is too vague, tests fail after 2 attempts, or merge conflicts need human judgment.
