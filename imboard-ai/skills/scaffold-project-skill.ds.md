---
name: 'scaffold-project-skill'
description: 'Scaffold a complete TypeScript project with CI, testing, linting, docs and worktree support. Use when user says ''scaffold project'', ''new typescript project'', ''bootstrap a repo'', ''create a new project''.'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Scaffold Project Skill'
  dossier.version: '1.0.0'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.objective: 'Trigger the scaffold-project dossier for a new TypeScript project'
  dossier.category: '["skills"]'
  dossier.tags: '["scaffold","typescript","boilerplate","skill"]'
  dossier.risk_level: 'medium'
  dossier.risk_factors: '["modifies_files"]'
  dossier.requires_approval: 'false'
  dossier.authors: '[{"name":"Yuval Dimnik"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"eccfdb5faf294c1460318c66cfdada62a16ddb58868c55b76f3e1d33a2d2577d"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"IDa5MVTZVWiIMC/mhgWvDqIxZNE/z4L0+UuFtKQSX9/rjkUsIbaF+ldugt98ymaK16XockHLiX/jRjczMR34Bw==","signed_at":"2026-10-07T14:09:14.239Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---

# Scaffold Project

## Inputs

Parse from the user's request:
- `project_name` (required): kebab-case name, used for `package.json` and the repo
- `project_dir` (required): absolute path to the project root
- `description` (required): one-line project description
- Optional: `github_org`, `license` (default MIT), `linter` (`biome` or `eslint`, default biome), `node_version` (default 22), `skip_worktrees`, `skip_github_repo`, `author_name`, `author_email`

## Steps

1. Ask for any required input that is missing
2. Run: `ai-dossier run imboard-ai/scaffold-project --pull` and pass the inputs through
3. Follow ALL phases in the dossier output. Do not skip any.
