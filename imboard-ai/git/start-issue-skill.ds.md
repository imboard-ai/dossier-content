---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "start-issue-skill",
  "title": "Start Issue Workflow",
  "version": "1.1.3",
  "protocol_version": "1.0",
  "status": "Deprecated",
  "objective": "Set up a GitHub issue for development with proper branch, worktree, and planning documentation",
  "description": "Set up a GitHub issue for development with branch, worktree, and planning doc. Use when user says \"start issue\", \"work on issue\"",
  "category": [
    "skills"
  ],
  "risk_level": "low",
  "requires_approval": false,
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "9e4490457d1d080430072c2847c52c2901dfbe67cc9d7ce0c550147b9ff384c0"
  },
  "relationships": {
    "alternatives": [
      {
        "dossier": "imboard-ai/skills/start-issue-skill",
        "when_to_use": "Always — this is the maintained location."
      }
    ]
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "602XDlnRRCq7DJgJ5Ug9M7GQi6ougH3eKRxFf6uUcCx6hFmZzflBQ1eVmMQubLQGtvfvuOD2OeDEY7zslXAABA==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-28T08:19:18.199Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---
> **Deprecated — moved to `imboard-ai/skills/start-issue-skill`.**
>
> Both paths installed to the same skill directory (`install-skill` keys on the
> basename), so whichever was installed last silently replaced the other. This copy
> is no longer maintained. Use `imboard-ai/skills/start-issue-skill`.


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
