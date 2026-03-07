---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "finish-issue-skill",
  "title": "Finish Issue Workflow",
  "version": "1.1.0",
  "status": "Stable",
  "objective": "Prepare a GitHub issue branch for PR with quality checks, cleanup, and review recommendations",
  "description": "Prepare a GitHub issue branch for PR with quality checks, cleanup, and review recommendations. Use when user says \"finish issue\", \"ready for PR\", \"prepare for review\", \"submit PR\", \"create PR\", \"finalize issue\", or \"wrap up\".",
  "authors": [
    {
      "name": "Dossier Community"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "60701af646e0bca5d4d78fe9097f9bb8ac330c058f20c16c50c15ef9435c7875"
  },
  "category": [
    "skills"
  ]
}
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
