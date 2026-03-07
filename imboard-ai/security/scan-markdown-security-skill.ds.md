---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "scan-markdown-security-skill",
  "title": "Markdown Security Scanner",
  "version": "1.0.0",
  "protocol_version": "1.0",
  "status": "Draft",
  "risk_level": "low",
  "requires_approval": false,
  "objective": "Scan markdown/dossier/skill files for security threats",
  "description": "Scan markdown files for prompt injection, agent hijacking, data exfiltration, XSS, malicious URLs, unicode attacks, and supply chain threats. Use when user says 'scan for security', 'audit skill', 'check dossier safety', 'is this markdown safe?'",
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "category": [
    "security"
  ],
  "tags": [
    "security",
    "scanning",
    "prompt-injection",
    "skill"
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "72ef0c516f1c2bdbceadc81908c9cfa16ea11f1f1694d4b9c12030902ecd2bf6"
  }
}
---

# Markdown Security Scanner

## Steps

1. Identify the target file(s) from the user's request
2. Run: `ai-dossier run imboard-ai/security/scan-markdown-security --pull`
3. Follow ALL scanning categories in the workflow output. Do not skip any.
4. Produce the full security scan report with findings table and risk assessment.
