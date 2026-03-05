---dossier
{
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "ec6a7a63e1aaed6f1eb7c39ba5ef98bfb73351aa24f3650c640a07a4413bd1b9"
  },
  "description": "Set up a GitHub issue for development with branch, worktree, and planning doc. Use when user says \"start issue\", \"work on issue\"",
  "name": "start-issue",
  "objective": "Set up a GitHub issue for development with proper branch, worktree, and planning documentation",
  "dossier_schema_version": "1.0.0",
  "status": "Draft",
  "title": "Start Issue Workflow",
  "version": "1.1.0"
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
   ai-dossier run imboard-ai/development/git/setup-issue-workflow
   ```
3. When prompted for issue number, provide the extracted number
4. Confirm successful setup with the user

## What This Creates

- A properly named branch (`feature/123-title` or `bug/123-title`)
- A git worktree for isolated development
- A `PLANNING.md` file to track implementation
