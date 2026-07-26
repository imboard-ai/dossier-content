---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "git-sync-skill",
  "title": "Git Sync",
  "version": "1.0.2",
  "protocol_version": "1.0",
  "status": "Draft",
  "objective": "Reconcile a repo's local state with origin: gitignore obvious junk, commit and push real work, surface ambiguous items",
  "description": "Triages uncommitted/unpushed state and syncs with GitHub. Use when arriving in a repo with dirty state, when the user says 'sync this repo', 'review uncommitted', 'commit pending', 'what's dirty here', 'push changes', or runs /git-sync.",
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
  "risk_level": "high",
  "requires_approval": false,
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "ed2b88726feadb111dae7f2b69f38e95eab2f80f075e1d8777f02d1f4d7e7df8"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "j7a30gAlMmUoH8LV/hxHIjIdgTURolSh3m4RAPqI8/EZvyTLrJWRPQxoT4vmZzTIgxZihe8RPXq3NGKW7ATEDg==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-26T12:47:30.201Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Git Sync

## Steps

1. Run: `ai-dossier run imboard-ai/git/git-sync --pull`
2. Follow the dossier's workflow exactly: pre-flight, snapshot, classify, present silent action plan, execute, ask on ambiguous items, final report.
3. Act autonomously on the silent-action bucket (auto-gitignore, auto-delete narrow scratch patterns, auto-commit single-purpose diffs, auto-push when ahead). Only ask the user about items the dossier classifies as ambiguous or sensitive.
4. Never bypass pre-commit hooks. Never force-push. Never auto-handle paths matching `.env*`, `*.pem`, `*.key`, `id_rsa*`, or anything under `secrets/`/`private/`/`credentials/`.
5. End with the dossier's final report — actions taken, items deferred, branch/remote state, worktree health.
