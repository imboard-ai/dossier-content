---
name: scan-markdown-security
title: Markdown Security Scanner
version: 1.1.0
schema_version: 1.0.0
status: draft
objective: Scan markdown/dossier/skill files for security threats
description: >
  Scan markdown files for prompt injection, agent hijacking, data exfiltration,
  XSS, malicious URLs, unicode attacks, and supply chain threats.
  Trigger on 'scan for security', 'audit skill', 'check dossier safety'.
authors:
  - name: Yuval Dimnik
---

# Markdown Security Scanner

When the user asks to scan a markdown, dossier, or skill file for security:

## Steps

1. Identify target file(s) from user request
2. Run the security scan dossier:
   ```bash
   ai-dossier run imboard-ai/security/scan-markdown-security
   ```
3. When prompted for target files, provide the identified paths

## Triggers

- "scan X for security", "audit this skill", "check dossier safety"
- "is this markdown safe?", "security scan", "check for injection"
- User provides an untrusted .ds.md or SKILL.md file

## Not Triggers

- General code security review (use security-code-reviewer agent)
- Runtime vulnerability scanning
- Dependency auditing
