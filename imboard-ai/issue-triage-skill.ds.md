---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "issue-triage-skill",
  "title": "Issue Triage",
  "version": "1.0.2",
  "protocol_version": "1.0",
  "status": "Draft",
  "objective": "Triage a GitHub issue to determine if it's ready for autonomous implementation",
  "description": "Assess and route issues: autonomous, plan-first, or not-ready. Use when user says 'triage issue', 'assess issue', 'check issue readiness'",
  "category": [
    "skills"
  ],
  "tags": [
    "github",
    "triage",
    "workflow",
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
    "hash": "4d6fad4ca37a11eaeddb8366d658f8e4525ba3b9a1a836066784de319cbde24c"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "Z0eIZWWpTlAWKDdzly5YVo46E3HMapBe9u/m+WzbF3KDv1xYpP5HyyS3UVd9fehNSKm/8inNFj+95PGg7TmiDQ==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-26T12:48:43.605Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Issue Triage

1. Extract the issue number from the user's request
2. Run: `ai-dossier run imboard-ai/issue-triage --pull`
3. Follow ALL phases in the workflow output. Do not skip any.
4. Be conservative in scoring — when borderline, prefer the more conservative lane (plan-first over autonomous, not-ready over plan-first).
