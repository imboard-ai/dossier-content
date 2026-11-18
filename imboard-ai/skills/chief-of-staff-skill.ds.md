---
name: 'chief-of-staff-skill'
description: 'Reduce the owner''s cognitive load across many parallel sessions: sweep all in-flight work, quietly finish the safe follow-through, and bring back ONE decision at a time as a 1/2/3 choice with a recommendation. Use when the user says ''where am I'', ''what needs me'', ''chief of staff'', ''I''m lost between sessions'', ''map my efforts'', ''what''s going on'', ''next decision'', or asks to triage/prioritize current work.'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Chief of Staff'
  dossier.version: '1.0.0'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.objective: 'Trigger the chief-of-staff dossier: sweep in-flight work, do safe follow-through, surface one owner decision at a time'
  dossier.category: '["skills"]'
  dossier.tags: '["owner","triage","cognitive-load","sessions","priorities","skill"]'
  dossier.risk_level: 'medium'
  dossier.requires_approval: 'false'
  dossier.authors: '[{"name":"Yuval Dimnik"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"458e9c56f5fe1e91bd8dd6788e2638aaec4903d79b2ae4100d96352a0dcf563b"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"s73xNAWY6g/IgnTAoOPAV9NedTuIg2XsDaLiVRggMKepVtkCWSEHmpK2umX3mYSlQnp9eIs2idLpWZyzOOstBw==","signed_at":"2026-11-18T21:02:26.928Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---

# Chief of Staff

Sweep everything in flight, quietly do the safe follow-through, and come back to the owner with **one item at a time**.

## Flags

Parse these from the user's request:
- `--dry`: sweep and surface only; change nothing (sets `authority=none`)
- `--max N`: stop after N surfaced items (sets `max_items`)

## Steps

1. Run: `ai-dossier run imboard-ai/git/chief-of-staff --pull`
2. Pass through `authority` and `max_items` from the flags above. Leave `repo` and `priorities_source` at their defaults unless the user names them.
3. Follow the dossier's owner-communication contract exactly: one decision per message, a numbered choice, a recommendation.
