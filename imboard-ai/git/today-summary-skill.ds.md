---
name: 'today-summary-skill'
description: 'Summarize today''s work based on git commits and file changes. Use when user asks "what did I do today", "daily summary", "today''s progress", or "end of day report".'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Today''s Work Summary'
  dossier.version: '1.0.3'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.objective: 'Generate a summary of today''s development work'
  dossier.category: '["skills"]'
  dossier.risk_level: 'low'
  dossier.requires_approval: 'false'
  dossier.authors: '[{"name":"Yuval Dimnik"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"abae09c36f7233084fad58be7c79bfa5dcebfcc40e0e5a7c50f15419375cbb91"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"OLqDLhhGi5tm3c6xQz5iDy3mdrqHUm2dkr7LV9rgiNop4RdVhwwtBePsOH4+Q+a8wmWYQ71jfNbbALHjH7MOAA==","signed_at":"2026-10-07T12:02:23.330Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
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
