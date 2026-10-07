---
name: 'issue-triage-skill'
description: 'Assess and route issues: autonomous, plan-first, or not-ready. Use when user says ''triage issue'', ''assess issue'', ''check issue readiness'''
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Issue Triage'
  dossier.version: '1.0.3'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.objective: 'Triage a GitHub issue to determine if it''s ready for autonomous implementation'
  dossier.category: '["skills"]'
  dossier.tags: '["github","triage","workflow","skill"]'
  dossier.risk_level: 'high'
  dossier.requires_approval: 'false'
  dossier.authors: '[{"name":"Yuval Dimnik"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"4d6fad4ca37a11eaeddb8366d658f8e4525ba3b9a1a836066784de319cbde24c"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"B/szSkmAvNRKaOqySgQD84DGexRyADPT4JubSTooZirLChaQOhnen8GqGin4SsKXgiGTP0Xm65gHGPrunrYrDg==","signed_at":"2026-10-07T12:03:18.015Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---

# Issue Triage

1. Extract the issue number from the user's request
2. Run: `ai-dossier run imboard-ai/issue-triage --pull`
3. Follow ALL phases in the workflow output. Do not skip any.
4. Be conservative in scoring — when borderline, prefer the more conservative lane (plan-first over autonomous, not-ready over plan-first).
