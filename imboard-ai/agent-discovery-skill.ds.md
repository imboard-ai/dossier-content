---
name: 'agent-discovery-skill'
description: 'Generate AGENTS.md and module catalog so external AI agents can discover project capabilities without full exploration. Use when user says ''make project discoverable'', ''agent discovery'', ''generate AGENTS.md'', ''document for agents'', ''create project manifest''.'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Agent Discovery Scaffold'
  dossier.version: '1.0.3'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.objective: 'Generate structured agent-discoverable documentation for the current project'
  dossier.category: '["skills"]'
  dossier.tags: '["agent-discovery","documentation","agents-md","skill"]'
  dossier.risk_level: 'low'
  dossier.requires_approval: 'false'
  dossier.authors: '[{"name":"Yuval Dimnik"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"407a003641081a4314507dc0c4d4334f636b654d533c3cc74a90b2cfc963b189"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"DW/uAqfCsCMWeUM+S8OBBfvbC/0rwn0FGydyUFHG3Gg7wNDWm6ykZE2dc4sUtLZFlkzB24gN7Ng7bk0FT7dLAg==","signed_at":"2026-10-07T11:56:11.342Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---

# Agent Discovery Scaffold

When the user wants to make a project discoverable by external AI agents:

## What This Does

Analyzes the current project and generates:
1. **AGENTS.md** — structured manifest of capabilities, reusable modules, and entry points
2. **docs/modules/** — per-module documentation for reusable components

This is different from CLAUDE.md (internal dev instructions). AGENTS.md is for external agents evaluating what this project offers.

## Steps

1. Identify the current project directory from the working directory
2. Run the scaffold dossier:
   ```bash
   ai-dossier run imboard-ai/docs/agent-discovery-scaffold
   ```
3. When prompted for `project_dir`, provide the current working directory
4. Default `output_format` is "both" (AGENTS.md + module catalog)
5. Review the generated files with the user

## Trigger Phrases

- "make this project discoverable"
- "generate AGENTS.md"
- "document for agents"
- "create project manifest"
- "agent discovery"
- "external agent documentation"
