---dossier
{
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "name": "agent-discovery-skill",
  "description": "Generate AGENTS.md and module catalog so external AI agents can discover project capabilities without full exploration. Use when user says 'make project discoverable', 'agent discovery', 'generate AGENTS.md', 'document for agents', 'create project manifest'.",
  "objective": "Generate structured agent-discoverable documentation for the current project",
  "dossier_schema_version": "1.0.0",
  "status": "Draft",
  "title": "Agent Discovery Scaffold",
  "version": "1.0.0",
  "category": [
    "skills"
  ],
  "tags": [
    "agent-discovery",
    "documentation",
    "agents-md",
    "skill"
  ]
}
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
