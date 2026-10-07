---
name: 'finish-issue-skill'
description: 'Prepare a GitHub issue branch for PR with quality checks, cleanup, and review recommendations. Use when user says "finish issue", "ready for PR", "prepare for review", "submit PR", "create PR", "finalize issue", or "wrap up".'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Finish Issue Workflow'
  dossier.version: '1.1.3'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Stable'
  dossier.objective: 'Prepare a GitHub issue branch for PR with quality checks, cleanup, and review recommendations'
  dossier.category: '["skills"]'
  dossier.risk_level: 'medium'
  dossier.requires_approval: 'false'
  dossier.authors: '[{"name":"Dossier Community"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"60701af646e0bca5d4d78fe9097f9bb8ac330c058f20c16c50c15ef9435c7875"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"2N7OZ4kM2RAjoFiaexpdrs4RbutMsHo3mnufvPUwIPiK4uXXZDtx+RzLpsIL80ZOBFprXoW/WApuZ/baBI+7Aw==","signed_at":"2026-10-07T11:58:28.376Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---

# Finish Issue Workflow

When the user wants to finalize their work on a GitHub issue and prepare a PR:

## Prerequisites

- ai-dossier CLI installed (`npm install -g @ai-dossier/cli`)
- On a feature/bug branch (not main/master)
- GitHub CLI (`gh`) authenticated

## Steps

1. Verify we're on a feature/bug branch (not main/master)
2. Run the finish workflow:
   ```bash
   ai-dossier run imboard-ai/git/finish-issue-workflow
   ```
3. Respond to prompts for each check category
4. Confirm successful PR creation with the user

## What This Does

- **Git prep**: Fetch latest, rebase onto main, check for conflicts
- **Security scans**: Detect secrets, hardcoded paths, files that should be gitignored
- **Code cleanup**: Find debug statements, commented code, unresolved TODOs
- **Project checks**: Run linter, formatter, type checker, tests
- **Review recommendations**: Suggest reviewers based on changed files (security, DB, ops, etc.)
- **PR creation**: Generate description from PLANNING.md + commits, link to issue, push and create PR
