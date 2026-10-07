---
name: 'start-issue'
description: 'Set up a GitHub issue for development with branch, worktree, and planning doc. Use when user says "start issue", "work on issue"'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Start Issue Workflow'
  dossier.version: '1.1.3'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.objective: 'Set up a GitHub issue for development with proper branch, worktree, and planning documentation'
  dossier.category: '["development"]'
  dossier.risk_level: 'low'
  dossier.requires_approval: 'false'
  dossier.authors: '[{"name":"Yuval Dimnik"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"e5ad16f83f7f441c98c85b8ca1c69f4c8b5dd6659918babad2e016b0073b9c23"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"CRagf7lR2Od/2p6n9vdkR3SJfdMf2nux2ZKVz8k1TEPPJGQTV+chSTgFBDR8ONwrDMpszyEpBH/8V4jwKDPjAA==","signed_at":"2026-10-07T12:02:05.429Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---

# Start Issue Workflow

When the user wants to start working on a GitHub issue:

## Prerequisites

Ensure ai-dossier CLI is installed:
```bash
npm install -g @ai-dossier/cli
```

If not installed, help the user install it first.

## Steps

1. Extract the issue number from their request
2. Run the setup workflow:
   ```bash
   ai-dossier run imboard-ai/git/setup-issue-workflow
   ```
3. When prompted for issue number, provide the extracted number
4. Confirm successful setup with the user

## What This Creates

- A properly named branch (`feature/123-title` or `bug/123-title`)
- A git worktree for isolated development
- A `PLANNING.md` file to track implementation
