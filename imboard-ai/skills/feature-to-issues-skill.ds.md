---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "feature-to-issues-skill",
  "title": "Feature to Issues",
  "version": "1.0.0",
  "status": "Draft",
  "risk_level": "medium",
  "objective": "Multi-agent feature development pipeline: problem signal → discovery → PRD → specs → GH issues → implementation → review loop",
  "description": "Orchestrate PM, UX/FE, and DB/BE agents to take a problem signal through discovery, PRD creation, spec generation, and GH issue decomposition. Use when user says 'feature to issues', 'plan feature', 'feature pipeline', 'new feature', 'feature development'",
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "category": [
    "skills"
  ],
  "tags": [
    "feature",
    "prd",
    "issues",
    "workflow",
    "multi-agent",
    "planning",
    "skill"
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "8850e5ab7e70761eda3cf6b5608d5ac3ec0f3a9ae4e25ecf9badd592bf105c03"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "lmisk2+W8b0TT/8z2jOFy18kXyjbzxXMTL2ud9LUXF4Fr8rArPIudFoeOpjv8IZq1ia38O5yXnaQls8jB5m6BA==",
    "public_key": "-----BEGIN PUBLIC KEY-----\nMCowBQYDK2VwAyEAm97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=\n-----END PUBLIC KEY-----\n",
    "signed_at": "2026-06-04T10:52:22.818Z",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Feature to Issues

Multi-agent feature development pipeline. Takes a problem signal and produces a complete feature folder with PRD, specs, and dependency-linked GitHub issues.

## Routing — Which Feature Workflow?

If the user's intent is ambiguous, clarify:

| Command | Behavior |
|---|---|
| **feature to issues "problem"** | Full pipeline: discover → discuss → PRD → specs → GH issues → implement → review |
| **guided cycle issue N** | Implement an EXISTING GH issue with plan review |
| **full cycle issue N** | Implement an EXISTING GH issue autonomously |

This skill works **upstream** of the issue cycles — it creates the GH issues that those cycles implement.

## Project Parameters

- **warmup_dossier**: *(project-specific — set per project, e.g., `imboard-ai/imboard/warm-worktree`)*

## Flags

Parse these from the user's request:
- `--skip-implementation`: Stop after GH issues are created (no implementation)
- `--mode <guided|full-cycle>`: Which issue workflow to use for implementation (default: guided)
- `--dir <path>`: Custom feature folder path

## Workflow

1. Extract the problem signal from the user's request (everything after the trigger phrase)
2. Run: `ai-dossier run imboard-ai/pm/feature-to-issues`
3. Pass through:
   - `problem_signal`: the extracted problem signal
   - `warmup_dossier`: from project parameters above (if set)
   - `skip_implementation`: true if `--skip-implementation` flag
   - `implementation_mode`: from `--mode` flag
   - `feature_dir`: from `--dir` flag
4. Follow ALL stages in the dossier output
5. The early stages (1-4) are conversation-heavy — interview the human, push back, evaluate, scope aggressively
6. The later stages (5-10) are more structured — produce artifacts, create issues, delegate implementation

## Prerequisites

Before running, verify:
- `ux-advisor` skill is available (check symlink: `ls -la ~/.claude/skills/ux-advisor`). If broken, warn the user.
- `pm-business-context` skill is available (optional — pipeline works without it)

## Resume Support

The dossier writes `STATUS.md` to the feature folder after each stage. If a session is interrupted:
1. The next invocation detects `STATUS.md`
2. Reads all existing artifacts to rebuild context
3. Offers to resume from the last completed stage

All work between stages is persisted to files. In-flight work within a stage is at risk, but draft artifacts are flushed periodically for long stages.
