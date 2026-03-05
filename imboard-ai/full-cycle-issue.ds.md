---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "full-cycle-issue",
  "title": "Full Cycle Issue",
  "version": "1.1.0",
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
    "hash": "f2f9505a13b55a0f9721a4df2bbebb87dc97bda41e68331cc7541a97792b0d33"
  }
}
---

# Full Cycle Issue

When the user wants to fully automate a small-to-medium GitHub issue:

## Steps

1. Extract the issue number from their request
2. Run the full-cycle workflow:
   ```bash
   ai-dossier run imboard-ai/development/full-cycle-issue
   ```
3. When prompted for issue number, provide the extracted number
4. The workflow handles everything: setup, implement, test, commit, push, PR, parallel review, merge

## Autonomy

Do NOT stop to ask yes/no questions. Only pause if:
- The issue is too vague to implement
- Tests fail after 2 fix attempts
- Merge conflicts need human judgment

Make all mechanical decisions (branch names, commit messages, PR descriptions) autonomously.
