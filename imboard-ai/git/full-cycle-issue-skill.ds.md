---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "full-cycle-issue-skill",
  "title": "Full Cycle Issue",
  "version": "1.3.0",
  "status": "Draft",
  "objective": "Take a GitHub issue from start to merged PR autonomously",
  "description": "Full autopilot: setup, implement, test, commit, push, PR, parallel review, merge. Use when user says 'full cycle issue', 'auto issue', 'autopilot issue'",
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "category": [
    "skills"
  ],
  "tags": [
    "github",
    "workflow",
    "autonomous",
    "skill"
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "fb3a75adc76c6459e1d2349f5429f80e915a8b571afcb374d86777b6f26b47f0"
  }
}
---

# Full Cycle Issue

1. Extract the issue number from the user's request
2. Run: `ai-dossier run imboard-ai/git/full-cycle-issue --pull`
3. Follow ALL phases in the workflow output. Do not skip any.
4. Do not stop to ask yes/no questions. Only pause if the issue is too vague, tests fail after 2 attempts, or merge conflicts need human judgment.
