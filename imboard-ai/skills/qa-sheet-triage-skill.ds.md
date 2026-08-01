---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "qa-sheet-triage-skill",
  "title": "QA Sheet Triage",
  "version": "1.0.0",
  "protocol_version": "1.0",
  "status": "Draft",
  "objective": "Turn a manual-QA findings spreadsheet into verified, clustered GitHub issues plus the detection-gap fixes that would have caught each bug automatically",
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
    "hash": "bffce7c010e1ad55f34c689c92d6fc4a0d1df0570c9546937a4a7eefab443f73"
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

## Steps

1. Extract the sheet reference from the user's request — a spreadsheet URL or ID, a local
   `.csv`/`.tsv` path, or pasted tabular text. If none was given, ask for it before running.
2. Run: `ai-dossier run imboard-ai/qa/sheet-triage --pull`
3. Pass through the `sheet` value and any flags above as the workflow's inputs.
4. Follow ALL phases in the workflow output. Do not skip any.
5. Do not stop to ask mechanical questions — cluster names, issue titles, and label choices
   are yours to decide. Pause only where the workflow explicitly says to: unreadable
   evidence, a genuine product decision, or a suspected duplicate of existing tracker work.
6. This workflow stops at filed issues. Do not implement fixes — that is a separate,
   deliberately-triggered workflow.
