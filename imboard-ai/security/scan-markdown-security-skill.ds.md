---
name: 'scan-markdown-security-skill'
description: 'Scan markdown files for prompt injection, agent hijacking, data exfiltration, XSS, malicious URLs, unicode attacks, and supply chain threats. Use when user says ''scan for security'', ''audit skill'', ''check dossier safety'', ''is this markdown safe?'''
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Markdown Security Scanner'
  dossier.version: '1.0.2'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.objective: 'Scan markdown/dossier/skill files for security threats'
  dossier.category: '["security"]'
  dossier.tags: '["security","scanning","prompt-injection","skill"]'
  dossier.risk_level: 'low'
  dossier.requires_approval: 'false'
  dossier.authors: '[{"name":"Yuval Dimnik"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"72ef0c516f1c2bdbceadc81908c9cfa16ea11f1f1694d4b9c12030902ecd2bf6"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"DqEXvEMQGQ0lmBtcOfYQjxf+yElaKoq6dXRrXrsFtkS3US6E2IjUiOx7gVG8NbhZRxKqd/hnC1lHgpxLf8ciDQ==","signed_at":"2026-10-07T12:04:58.914Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---

# Markdown Security Scanner

## Steps

1. Identify the target file(s) from the user's request
2. Run: `ai-dossier run imboard-ai/security/scan-markdown-security --pull`
3. Follow ALL scanning categories in the workflow output. Do not skip any.
4. Produce the full security scan report with findings table and risk assessment.
