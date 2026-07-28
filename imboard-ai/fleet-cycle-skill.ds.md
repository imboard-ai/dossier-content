---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "fleet-cycle-skill",
  "title": "Fleet Cycle",
  "version": "1.0.3",
  "protocol_version": "1.0",
  "status": "Deprecated",
  "objective": "Take a SET of GitHub issues to merged PRs via dependency-aware waves of background full-cycle runs",
  "description": "Orchestrate multiple issues at once. Builds a dependency-aware wave plan and dispatches full-cycle-issue across background agents — parallel where safe, serial where dependent. Use when the user says 'fleet cycle', 'full cycle issues 1,2,3', 'full cycle issues 1..9', 'batch issues', 'map these issues and run them', 'run these issues in parallel/serial', or gives a LIST or RANGE of issues to take to merged PRs.",
  "category": [
    "skills"
  ],
  "tags": [
    "github",
    "workflow",
    "autonomous",
    "orchestration",
    "batch",
    "parallel",
    "fleet",
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
    "hash": "fbc52bc2d0eed0b1788b15be743f17af9da19edeb06773e1dbafedcecbbc9c8c"
  },
  "relationships": {
    "alternatives": [
      {
        "dossier": "imboard-ai/skills/fleet-cycle-skill",
        "when_to_use": "Always — this is the maintained location."
      }
    ]
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "BcT2VEVcHKbd1T7MMaCclNyZLBinYPAXo0uZxmI2gmk1wz7uNOPBDGEDHKTi3z90Yhvf6IGNDhskUd8GMttxCA==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-28T08:19:11.396Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---
> **Deprecated — moved to `imboard-ai/skills/fleet-cycle-skill`.**
>
> Both paths installed to the same skill directory (`install-skill` keys on the
> basename), so whichever was installed last silently replaced the other. This copy
> is no longer maintained. Use `imboard-ai/skills/fleet-cycle-skill`.


# Fleet Cycle

Orchestrate a **set** of GitHub issues to merged PRs. This is the batch layer above `full-cycle-issue`: it plans dependencies, then dispatches one full-cycle run per issue across background agents.

## Project Parameters

- **warmup_dossier**: `imboard-ai/git/warm-worktree` (generic; auto-detects package manager). For the imboard monorepo's pnpm + SSM path, override with `imboard-ai/imboard/warm-worktree`.

## Flags

Parse these from the user's request:
- `--max-parallel N`: max concurrent runs within a wave (default 3)
- `--serial`: force one issue at a time, ascending order (sets `mode=serial`)
- `--parallel`: ignore dependencies, run all at once — only for known-independent issues (sets `mode=parallel`)
- `--base <branch>`: default base branch for issues that don't declare their own

## Steps

1. Extract the **issue set** from the user's request — a list (`1,2,3`), a range (`1..9`), or mixed (`1,2,5..8`).
2. Run: `ai-dossier run imboard-ai/git/fleet-cycle --pull`
3. Pass through: the resolved `issues`, plus `max_parallel`, `mode`, `warmup_dossier`, and `base_branch` from the flags above.
4. Follow ALL phases in the workflow output: resolve set → build dependency graph → compute wave plan → dispatch & supervise → aggregate report. Do not skip any.
5. **Plan, then auto-run.** Present the wave plan, then dispatch automatically — do not wait for approval. Pause only for an invalid issue, a true dependency cycle, or an unavoidably long fully-serial chain.
6. Each issue is handled by a background `full-cycle-issue` run; supervise them and block a failed issue's dependents.
