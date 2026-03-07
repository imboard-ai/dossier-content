---dossier
{
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "description": "Set up a GitHub issue for development with branch, worktree, and planning doc. Use when user says \"start issue\", \"work on issue\"",
  "name": "start-issue-skill",
  "objective": "Set up a GitHub issue for development with proper branch, worktree, and planning documentation",
  "dossier_schema_version": "1.0.0",
  "status": "Draft",
  "title": "Start Issue Workflow",
  "version": "1.1.0",
  "checksum": {
    "algorithm": "sha256",
    "hash": "e5ad16f83f7f441c98c85b8ca1c69f4c8b5dd6659918babad2e016b0073b9c23"
  },
  "category": [
    "skills"
  ]
}
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
