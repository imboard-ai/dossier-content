---
name: 'worktree-pool-skill'
description: 'Manage a pool of pre-warmed git worktrees so an issue starts in about 2 seconds instead of a 3-5 minute cold setup. Use when user says ''worktree pool'', ''warm worktrees'', ''claim a worktree'', ''pre-warm worktrees''.'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Worktree Pool Skill'
  dossier.version: '1.0.0'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.objective: 'Trigger the worktree-pool dossier to init, replenish, claim and inspect a pool of warm worktrees'
  dossier.category: '["skills"]'
  dossier.tags: '["worktree","git","pre-warm","skill"]'
  dossier.risk_level: 'low'
  dossier.risk_factors: '["modifies_files"]'
  dossier.requires_approval: 'false'
  dossier.authors: '[{"name":"Yuval Dimnik"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"1cfff199944a0cb677d24cd0dc298862ba0cd1f5c6d3960b8561488bb5105df5"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"3sgVyrjLhfwqkAFw5CFDyqvF3iecXZLPrT+deq4GgOp+U1lLxsvQXkXRBqc5NoEdiNrIMOlYdf0hWySmUkHfBA==","signed_at":"2026-10-07T14:09:16.692Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---

# Worktree Pool

## Inputs

Parse from the user's request:
- the action: `init` (once per project), `replenish` (with a count), `claim` (with an issue number and branch), or `status`
- `issue` and `branch` when claiming

## Steps

1. Run: `ai-dossier run imboard-ai/git/worktree-pool --pull`
2. Follow the dossier output for the requested action
3. Report the claimed worktree path, or the pool status
