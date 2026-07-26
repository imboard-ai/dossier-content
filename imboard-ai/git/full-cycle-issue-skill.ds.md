---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "full-cycle-issue-skill",
  "title": "Full Cycle Issue",
  "version": "1.3.2",
  "protocol_version": "1.0",
  "status": "Draft",
  "objective": "Take a GitHub issue from start to merged PR autonomously",
  "description": "Full autopilot: setup, implement, test, commit, push, PR, parallel review, merge. Use when user says 'full cycle issue', 'auto issue', 'autopilot issue'",
  "category": [
    "skills"
  ],
  "tags": [
    "github",
    "workflow",
    "autonomous",
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
    "hash": "fb3a75adc76c6459e1d2349f5429f80e915a8b571afcb374d86777b6f26b47f0"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "FZcpF+bBd7FCj1tQxDuorA2gGvZZkMnZIbfYX3FiY4bdfBvAdt/aYMfENlkXNp4qklK6JTQGQtbNbdRUdVfGDA==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-26T12:47:20.822Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Full Cycle Issue

1. Extract the issue number from the user's request
2. Run: `ai-dossier run imboard-ai/git/full-cycle-issue --pull`
3. Follow ALL phases in the workflow output. Do not skip any.
4. Do not stop to ask yes/no questions. Only pause if the issue is too vague, tests fail after 2 attempts, or merge conflicts need human judgment.
