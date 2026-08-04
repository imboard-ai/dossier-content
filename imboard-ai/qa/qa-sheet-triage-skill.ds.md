---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "qa-sheet-triage-skill",
  "title": "QA Sheet Triage",
  "version": "1.1.0",
  "protocol_version": "1.0",
  "status": "Draft",
  "objective": "Turn a manual-QA findings spreadsheet into verified, clustered GitHub issues plus the detection-gap fixes that would have caught each bug automatically, graded for autonomous readiness and ordered into an execution plan",
  "description": "Triage a manual-QA findings sheet into GitHub issues and prevention work. Use when user says 'triage QA sheet', 'QA findings', 'manual QA results', 'QA round', or hands over a spreadsheet of bugs",
  "risk_level": "medium",
  "requires_approval": false,
  "category": [
    "testing"
  ],
  "tags": [
    "qa",
    "triage",
    "github",
    "workflow",
    "skill"
  ],
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "040c31caa1675d7d81d9e6f8d2202ff7b15e0aa12b49548e0e1a4cef0bb3f303"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "3cfT+oAoDIWn828PE68UMLKqX6a9vERpgwWYHWTpfaY/QI4wFArrvpYn7u+QPPlYJ4z4w1wwkeL5THZyJ9QtBA==",
    "public_key": "-----BEGIN PUBLIC KEY-----\nMCowBQYDK2VwAyEAT5MH6NyHt3zBur6eq+EVSNOA2AZbuSRpov+/BRFzLnY=\n-----END PUBLIC KEY-----\n",
    "signed_at": "2026-08-04T05:55:56.656Z",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# QA Sheet Triage

## Flags

Parse these from the user's request:

- `--dry-run`: run every investigation phase and report, but create no issues and no companion sheet
- `--only <IDs>`: process only these finding IDs (e.g. `--only BUG-003,BUG-004`)
- `--no-writeback`: file issues but create no companion sheet
- `--checkpoint`: always stop for review after the Phase 1 surface review
- `--repo <owner/name>`: target repository for issues (defaults to the current repo's origin)
- `--ready-label <name>`: label for issues judged ready for autonomous full-cycle execution
  (default `ready:full-cycle`)

## Steps

1. Extract the sheet reference from the user's request — a spreadsheet URL or ID, a local
   `.csv`/`.tsv` path, or pasted tabular text. If none was given, ask for it before running.
2. Run: `ai-dossier run imboard-ai/qa/sheet-triage --pull`
3. Pass through the `sheet` value and any flags above as the workflow's inputs.
4. Follow ALL phases in the workflow output. Do not skip any.
5. Do not stop to ask mechanical questions — cluster names, issue titles, and label choices
   are yours to decide. Pause only where the workflow explicitly says to: unreadable
   evidence, a genuine product decision, or a suspected duplicate of existing tracker work.
6. The run ends with a readiness verdict on every filed issue and a wave-ordered execution
   plan for the ready ones. **Present the plan; do not run it.** Dispatching it via
   `fleet-cycle` is a separate, deliberately-triggered act, and so is implementing any fix.
7. Write nothing into the repository working tree. The cycle log belongs at
   `~/.dossier/logs/qa-sheet-triage/<owner>-<repo>/TRIAGE-CYCLE-LOG.md`.
