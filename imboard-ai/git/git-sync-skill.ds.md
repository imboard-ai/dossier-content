---dossier
{
  "dossier_schema_version": "1.0.0",
  "title": "Git Sync",
  "version": "1.0.0",
  "status": "Draft",
  "objective": "Reconcile a repo's local state with origin: gitignore obvious junk, commit and push real work, surface ambiguous items",
  "category": [
    "skills"
  ],
  "tags": [
    "git",
    "github",
    "sync",
    "workflow",
    "skill"
  ],
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "description": "Triages uncommitted/unpushed state and syncs with GitHub. Use when arriving in a repo with dirty state, when the user says 'sync this repo', 'review uncommitted', 'commit pending', 'what's dirty here', 'push changes', or runs /git-sync.",
  "name": "git-sync-skill"
}
---

# Git Sync

## Steps

1. Run: `ai-dossier run imboard-ai/git/git-sync --pull`
2. Follow the dossier's workflow exactly: pre-flight, snapshot, classify, present silent action plan, execute, ask on ambiguous items, final report.
3. Act autonomously on the silent-action bucket (auto-gitignore, auto-delete narrow scratch patterns, auto-commit single-purpose diffs, auto-push when ahead). Only ask the user about items the dossier classifies as ambiguous or sensitive.
4. Never bypass pre-commit hooks. Never force-push. Never auto-handle paths matching `.env*`, `*.pem`, `*.key`, `id_rsa*`, or anything under `secrets/`/`private/`/`credentials/`.
5. End with the dossier's final report — actions taken, items deferred, branch/remote state, worktree health.
