---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "full-cycle-issue-skill",
  "title": "Full Cycle Issue",
  "version": "1.3.3",
  "protocol_version": "1.0",
  "status": "Deprecated",
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
    "hash": "be52df96c9a1514f3a2a14f3c5e36e336ab87f454f2a1c22e048b3e98eccb068"
  },
  "relationships": {
    "alternatives": [
      {
        "dossier": "imboard-ai/skills/full-cycle-issue-skill",
        "when_to_use": "Always — this is the maintained location."
      }
    ]
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "fC4uqg6xs3rl0t2SS+el+dFOWuBRQXO4f6GJzYlO4wfD/0OQttvJXZOCUTGiKtNm7S/7tw1wOlvoPmRXpIV/Bg==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-28T08:21:11.089Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---
> **Deprecated — moved to `imboard-ai/skills/full-cycle-issue-skill`.**
>
> Both paths installed to the same skill directory (`install-skill` keys on the
> basename), so whichever was installed last silently replaced the other. This copy
> is no longer maintained. Use `imboard-ai/skills/full-cycle-issue-skill`.


# Full Cycle Issue

1. Extract the issue number from the user's request
2. Run: `ai-dossier run imboard-ai/git/full-cycle-issue --pull`
3. Follow ALL phases in the workflow output. Do not skip any.
4. Do not stop to ask yes/no questions. Only pause if the issue is too vague, tests fail after 2 attempts, or merge conflicts need human judgment.
