---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "today-summary-skill",
  "title": "Today's Work Summary",
  "version": "1.0.2",
  "protocol_version": "1.0",
  "status": "Draft",
  "objective": "Generate a summary of today's development work",
  "description": "Summarize today's work based on git commits and file changes. Use when user asks \"what did I do today\", \"daily summary\", \"today's progress\", or \"end of day report\".",
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
    "hash": "61f6f93fd2271af2c845916549958626410da91bd63a5c6d67c2604230efcbe6"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "+aIzOKL0cuQzIgEqQdxX/CpaRToIafRr1nB92Z3pzPgD4CLOOHWzXUJt4TeR7u2ufHmq+enQI0pxA83HKoCpBQ==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-26T12:48:24.191Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Today's Work Summary

When the user asks for a summary of today's work:

## Steps

1. Get today's commits:
   ```bash
   git log --since="midnight" --oneline --all
   ```

2. Get file change statistics:
   ```bash
   git diff --stat $(git log --since="midnight" --format=%H | tail -1)^..HEAD
   ```

3. Summarize:
   - Number of commits
   - Files modified/added/deleted
   - Key changes by category (features, fixes, refactoring, docs)
   - Time span of work (first to last commit)

## Output Format

Present a concise summary like:
- "Today you made X commits between HH:MM and HH:MM"
- "Main areas: [list affected directories/components]"
- "Highlights: [brief description of significant changes]"