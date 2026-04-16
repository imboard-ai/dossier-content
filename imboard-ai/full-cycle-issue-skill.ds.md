---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "full-cycle-issue-skill",
  "title": "Full Cycle Issue",
  "version": "1.6.0",
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
    "hash": "e987430caa1f7235e771a8ec6549371283446995a3fbd6d7dca1eea810cf7ac0"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "e0aE2YMqGkGggDK8/vml9rLqI78ZE92UI510o1YpmLpNVn9L95O6VHpPWYQxkrQXHFSdCRN2uzNBqZkivZqABg==",
    "public_key": "-----BEGIN PUBLIC KEY-----\nMCowBQYDK2VwAyEANvCyj7RG85NwvvB5AozhC8Oep2DnwY4X3YC1fY/8E9E=\n-----END PUBLIC KEY-----\n",
    "signed_at": "2026-04-16T13:32:23.753Z",
    "signed_by": "(not specified)"
  }
}
---

# Full Cycle Issue

## Project Parameters

- **warmup_dossier**: `imboard-ai/git/warm-worktree` (generic; auto-detects package manager from lockfiles). For the imboard monorepo's faster pnpm + SSM path, override with `imboard-ai/imboard/warm-worktree`.

## Flags

Parse these from the user's request:
- `--base <branch>`: Override the target branch (bypasses issue body parsing)

## Steps

1. Extract the issue number from the user's request
2. Run: `ai-dossier run imboard-ai/git/full-cycle-issue --pull`
3. The workflow will use the `warmup_dossier` parameter above for worktree warmup (auto-detects package manager, copies .env files, installs deps, verifies build/tests/servers)
4. If `--base` flag was provided, pass the base_branch parameter to the workflow
5. Follow ALL phases in the workflow output. Do not skip any.
6. Do not stop to ask yes/no questions. Only pause if the issue is too vague, tests fail after 2 attempts, or merge conflicts need human judgment.
