---dossier
{
  "dossier_schema_version": "1.0.0",
  "protocol_version": "1.0",
  "name": "autopilot-loop-skill",
  "title": "Autopilot Loop",
  "version": "1.0.0",
  "status": "Draft",
  "objective": "Run a repository's backlog unattended in budget-gated, self-paced cycles with worker agents, orchestrator review, and a pinned log/steering issue",
  "description": "Unattended backlog orchestration: each self-paced cycle checks a usage budget, picks the highest-value issues, dispatches cheaper worker agents to green-CI PRs, reviews and merges them as the orchestrator, verifies releases, and logs to a pinned GitHub issue the owner steers from. Use when the user says 'autopilot', 'run the backlog while I'm away', 'loop on the issues', 'progress the repo on your own', 'keep working until X% usage', or asks for an unattended/self-paced loop over a repo's issues.",
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "category": [
    "skills"
  ],
  "tags": [
    "github",
    "autonomous",
    "orchestration",
    "loop",
    "autopilot",
    "skill"
  ],
  "risk_level": "high",
  "requires_approval": false,
  "checksum": {
    "algorithm": "sha256",
    "hash": "8c773beb0e65e1be9c8ec3caff12f8ba3cd78af7a6fab27184eab8fe80956d87"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "Ssq1ebPp5Nw1GTr2Ofo0BzcqjvZ+rOBUqTWkd5OZEvspdIWtKTsmKyH3A2iawJt38PWMsbx3ys61tz3Hl4LLBQ==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-09-29T15:12:47.077Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Autopilot Loop

Progress a repository's backlog unattended. This is the loop layer above `full-cycle-issue`: budget gate, pick, dispatch workers, review and merge, verify, log, repeat.

## Flags

Parse these from the user's request:
- `--budget N`: stop at N% weekly usage (default 40)
- `--authority pr|merge|release`: what may happen without asking (default merge)
- `--workers N`: max parallel worker units (default 2)
- `--worker-model <model>`: model for workers (default sonnet)
- `--log-issue N`: reuse an existing log/steering issue
- `--lessons-every H`: lessons-learned review interval in hours (default 6)

## Steps

1. Confirm once, only if unstated: repo, budget cap, authority, worker model, and the usage command. Offer recommended defaults, then never ask again.
2. Run: `ai-dossier run imboard-ai/git/autopilot-loop --pull`
3. Pass through: `repo`, `budget_cap_percent`, `authority`, `max_parallel_workers`, `worker_model`, `log_issue`, `lessons_interval_hours`, `usage_command`.
4. Do Phase 0 (kickoff), then start a self-paced loop (e.g. `/loop` dynamic mode) whose prompt runs Phases 1–6 each cycle and Phase 7 at the interval.
5. Follow the dossier's **Rules** literally. They are distilled from real failures.
