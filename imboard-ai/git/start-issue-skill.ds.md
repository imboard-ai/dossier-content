---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "start-issue-skill",
  "title": "Start Issue Workflow",
  "version": "1.1.2",
  "protocol_version": "1.0",
  "status": "Draft",
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
    "hash": "e5ad16f83f7f441c98c85b8ca1c69f4c8b5dd6659918babad2e016b0073b9c23"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "YV1k27ch+EIS3SVUZReirK4kazip3wz0SV3H/5d0yhIE27xJfOL5ZtVTkig0k0GLTo5S3vuUE12k6vtqUEb6DA==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-26T12:48:17.748Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
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
