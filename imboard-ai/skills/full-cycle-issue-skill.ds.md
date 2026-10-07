---
name: 'full-cycle-issue-skill'
description: 'Full autopilot: gate, setup, plan, implement, test, review, commit, push, PR, merge, rich report. Use when user says ''full cycle issue'', ''auto issue'', ''autopilot issue'', ''fire and forget'''
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Full Cycle Issue'
  dossier.version: '1.6.4'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.objective: 'Take a GitHub issue from start to merged PR autonomously'
  dossier.category: '["skills"]'
  dossier.tags: '["github","workflow","autonomous","skill"]'
  dossier.risk_level: 'high'
  dossier.requires_approval: 'false'
  dossier.authors: '[{"name":"Yuval Dimnik"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"5ce3dd57756d05ada556e85e169bc89c4004b9b61fd1f58daccc31848636d93f"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"oA/51BPbCMWc0W5+dvKiHs3ZQgaswZrlc0SwtykNfT0VW+d2mS0a+5jrcx792cdgH1KRv+T2Sh6Z2yyLeNPEBA==","signed_at":"2026-10-07T12:06:00.530Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---

# Full Cycle Issue

## Project Parameters

- **warmup_dossier**: `imboard-ai/git/warm-worktree` (generic; auto-detects package manager from lockfiles). For the imboard monorepo's faster pnpm + SSM path, override with `imboard-ai/imboard/warm-worktree-pnpm-ssm`.

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
