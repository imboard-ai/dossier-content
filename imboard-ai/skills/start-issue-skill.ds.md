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
  "version": "1.3.2",
  "checksum": {
    "algorithm": "sha256",
    "hash": "47ff1751e2251eb3e26e1a2f587bf7b88441b70a236bfd579119aff972501484"
  },
  "category": [
    "skills"
  ],
  "protocol_version": "1.0",
  "risk_level": "medium",
  "requires_approval": false,
  "signature": {
    "algorithm": "ed25519",
    "signature": "H92c3g/c7gLJs9AIZXFjOc+uFGMUxG2GOOdZaJCvUAh6C4LmLTiPmijSAAFvi6T0EH3sx5aZY/7F0CJHjs/QBQ==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-28T08:22:20.447Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Start Issue Workflow

When the user wants to start working on a GitHub issue:

## Project Parameters

- **warmup_dossier**: `imboard-ai/imboard/warm-worktree-pnpm-ssm`

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
