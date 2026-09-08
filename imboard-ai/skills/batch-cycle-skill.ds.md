---dossier
{
  "dossier_schema_version": "1.0.0",
  "protocol_version": "1.0",
  "name": "batch-cycle-skill",
  "title": "Batch Cycle",
  "version": "1.0.0",
  "status": "Draft",
  "objective": "Take a SET of GitHub issues to ONE pull request, paying the repo's expensive verification once for all of them instead of once each",
  "description": "Batch several issues into ONE PR with ONE expensive verification run. Each issue gets its own agent and worktree off a shared integration branch; a parent orchestrator merges them, runs the repo's full gate once, repairs what breaks, and ships a single PR. Use when the user says 'batch cycle', 'batch these issues into one PR', 'run these issues as a batch', 'one PR for these issues', or asks to avoid paying CI/verification per issue. NOT for when each issue needs its own PR — that is fleet-cycle.",
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
    "workflow",
    "autonomous",
    "orchestration",
    "batch",
    "integration-branch",
    "verification",
    "skill"
  ],
  "risk_level": "high",
  "requires_approval": false,
  "checksum": {
    "algorithm": "sha256",
    "hash": "7a26ba75abb6abd170ea0471d484d5f78b8c934f907516796eb7cc819f3c5649"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "lXb0c0ZPCYaUjyiVTUEdvABfsj0a8sD7kFdbwd4HRLUBel07I+baf2//5rfGHic79TnZcjJneqLwuOTV/Ac7AQ==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-09-08T22:44:57.031Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Batch Cycle

Take a **set** of issues to **one** pull request. Each issue is implemented independently, in parallel; the repo's expensive verification runs **once** over the combined result.

## When to use this, and when not to

| | Batch Cycle | Fleet Cycle |
|---|---|---|
| Result | **one PR** closing N issues | **N PRs**, one per issue |
| Expensive verification | **once** | once per issue |
| Use when | the repo's gate is slow and you are shipping several issues | each issue needs to land and be revertable on its own |

**The whole point is amortization.** If the repo's verification gate is cheap, or the issues must ship independently, use `fleet-cycle-skill` instead.

Measured on a repo whose local gate takes ~50–90 minutes: 3 issues batched took 81 min against 157–267 min separately; 6 issues took 94 min against 312–534 min. The gate grows sublinearly, so **larger batches are better** — the fixed floor is paid once either way.

## The pieces

| Layer | What it is |
|---|---|
| `imboard-ai/git/batch-issues-preparation` | classify the set, compose the batch, enqueue |
| `ai-dossier sched` | creates the integration branch, dispatches members in parallel worktrees |
| `imboard-ai/git/member-cycle` | one agent per issue: implement, test by relevance, hand over |
| `imboard-ai/git/batch-integrate` | the parent: merge, verify once, repair, ship one PR |

**Do not use `imboard-ai/git/batch-issues`** — it predates this model and orchestrates a different thing entirely. It sorts first in a registry search for "batch"; it is the wrong one.

## Steps

### 1. Preflight the repo — before dispatching anything

**Check the capability manifest.** `ai-dossier cap list` in the target repo. A repo with no `.dossier/automation/manifest.yaml` reports no capabilities, and undeclared gate capabilities are *skipped* — so members run with **no gate at all**, silently. Tell the operator before dispatching, not after.

**Check what the expensive gate actually costs here.** If the repo's full verification is a few minutes, batching buys little; say so and suggest fleet-cycle.

### 2. Screen the set for readiness

Cheap checks that prevent expensive failures. For each issue, read the **body**, not the labels:

- **Does every artifact it names exist on the base branch?** An issue saying "migrate onto the hook extracted by #N" depends on #N — whether or not it says "Depends on". Verify the symbol exists; do not trust the prose.
- **Does the body enumerate a countable work list?** Count it. A "documentation" issue can be thousands of lines.
- **Is it assigned or in progress?** Someone may already be on it.
- **Is it a tracker or a decision?** A body listing many independent findings, or headed "Decision needed", has no stopping point for an agent.
- **Does it mutate or delete data?** Those need independent revert granularity — keep them out of a shared PR.

Drop what fails, and say why. A dropped issue costs nothing; a member forcing work against a missing dependency costs an agent run.

### 3. Compose the batch

**Prefer a mixed cohort.** Batch value is a function of the **union of members' affected scopes**, not the member count: once one member triggers the repo's expensive stage, every member added after it rides along at almost no extra gate cost. A batch of issues that all avoid the expensive stage amortizes almost nothing.

**Keep slices of one designed sequence out of the same batch** — PR1/PR2/PR3 of a feature are not independent and will conflict by construction. Either batch one of them, or expect to resolve the collision.

**Do not size the batch by predicted diff.** Diff size predicts neither cost nor conflict: measured members have run 92 turns for a net −29 lines, and 59 turns for +193.

### 4. Dispatch and integrate

The scheduler creates the integration branch and dispatches one `member-cycle` agent per issue, in parallel, each in its own worktree. When all members have landed or handed back, run `batch-integrate`.

### 5. Ship

One PR, **rebase-merged, never squashed** — per-issue commits carry the attribution the model depends on.

## What to tell the operator

- Which issues were dropped in screening, and why
- Any member that handed back rather than implementing — this is a valued outcome, not a failure
- Every batch-level repair the parent made, and its cause
- If the gate failed on infrastructure rather than code: what was retried, and what remains unverified

## Notes

- A batch that outlives another batch's merge inherits its changes; the base can move several commits during one batch's lifetime on an active repo. The parent merges the base again before shipping.
- Batch verification often depends on a **shared** resource (a test-database pool, a runner). It may be shared across machines, not just processes. Concurrent batches can starve each other, and the symptom looks exactly like a member regression. `batch-integrate` handles this via its four-way verdict — but if you can, do not run two batch gates at once.
