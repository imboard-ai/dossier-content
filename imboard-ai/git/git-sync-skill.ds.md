---
name: 'git-sync-skill'
description: 'Triages uncommitted/unpushed state and syncs with GitHub. Use when arriving in a repo with dirty state, when the user says ''sync this repo'', ''review uncommitted'', ''commit pending'', ''what''s dirty here'', ''push changes'', or runs /git-sync.'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Git Sync'
  dossier.version: '1.0.3'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.objective: 'Reconcile a repo''s local state with origin: gitignore obvious junk, commit and push real work, surface ambiguous items'
  dossier.category: '["skills"]'
  dossier.tags: '["git","github","sync","workflow","skill"]'
  dossier.risk_level: 'high'
  dossier.requires_approval: 'false'
  dossier.authors: '[{"name":"Yuval Dimnik"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"ed2b88726feadb111dae7f2b69f38e95eab2f80f075e1d8777f02d1f4d7e7df8"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"wlMy7IWBaM9WXne+genBWpPIUcdqp4YDAjo9ZjlK0HUGwsUN97UeGbrY1D4hlm13ut9mgFw80+QMz+9Ll8W4BQ==","signed_at":"2026-10-07T11:59:19.951Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---

# Git Sync

## Steps

1. Run: `ai-dossier run imboard-ai/git/git-sync --pull`
2. Follow the dossier's workflow exactly: pre-flight, snapshot, classify, present silent action plan, execute, ask on ambiguous items, final report.
3. Act autonomously on the silent-action bucket (auto-gitignore, auto-delete narrow scratch patterns, auto-commit single-purpose diffs, auto-push when ahead). Only ask the user about items the dossier classifies as ambiguous or sensitive.
4. Never bypass pre-commit hooks. Never force-push. Never auto-handle paths matching `.env*`, `*.pem`, `*.key`, `id_rsa*`, or anything under `secrets/`/`private/`/`credentials/`.
5. End with the dossier's final report — actions taken, items deferred, branch/remote state, worktree health.
