---dossier
{
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "description": "Setup only: gate, branch, worktree, and rich planning doc. Use when user says 'start issue', 'setup issue', 'prepare issue', 'work on issue'",
  "name": "start-issue-skill",
  "objective": "Set up a GitHub issue for development with safety gate, proper branch, worktree, and rich planning documentation",
  "dossier_schema_version": "1.0.0",
  "status": "Draft",
  "title": "Start Issue Workflow",
  "version": "1.3.0",
  "checksum": {
    "algorithm": "sha256",
    "hash": "e8e7977e01910753c33c385e83c8ce7af409717f7f02afbe92829bcbd5fc3264"
  },
  "category": [
    "skills"
  ]
}
---

# Start Issue Workflow

When the user wants to start working on a GitHub issue:

## Project Parameters

- **warmup_dossier**: `imboard-ai/imboard/warm-worktree`

## Flags

Parse these from the user's request:
- `--base <branch>`: Override the target branch (bypasses issue body parsing)

## Steps

1. Extract the issue number from their request
2. **Gate**: Run `ai-dossier run imboard-ai/git/gate-issue` — safety check for hard blocks
3. If the gate fails, stop and report the reason
4. Note the `base_branch` from the gate output (or use `--base` flag if provided)
5. **Setup**: Run `ai-dossier run imboard-ai/git/setup-issue-workflow`
   - Pass through `warmup_dossier` and `base_branch` parameters
6. `cd` into the worktree directory
7. **Plan**: Run `ai-dossier run imboard-ai/git/plan-issue`
   - Pass through the issue number, base_branch, and worktree path
8. Present the generated `PLANNING-{number}-{slug}.md` to the user

## What This Creates

- A properly named branch (`feature/123-title` or `bug/123-title`)
- A git worktree for isolated development
- A rich `PLANNING-{number}-{slug}.md` with problem analysis, approach, files to modify, risk areas, and test strategy
