---dossier
{
  "dossier_schema_version": "1.0.0",
  "protocol_version": "1.0",
  "name": "batch-cycle-skill",
  "title": "Batch Cycle",
  "version": "1.3.0",
  "status": "Draft",
  "last_updated": "2026-09-12",
  "objective": "Take a SET of GitHub issues to ONE pull request, paying the repo's expensive verification once for all of them instead of once each",
  "description": "Batch several issues into ONE PR with ONE expensive verification run. Each issue gets its own agent and worktree off a shared integration branch; a parent orchestrator merges them, runs the repo's full gate once, repairs what breaks, and ships a single PR. Use when the user says 'batch cycle', 'batch these issues into one PR', 'run these issues as a batch', 'one PR for these issues', or asks to avoid paying CI/verification per issue. NOT for when each issue needs its own PR — that is fleet-cycle.",
  "inputs": {
    "optional": [
      {
        "name": "dispatch_profile",
        "description": "Configured scheduler dispatch profile selected from the operator's stated provider family; omit only when no profiles are configured and the default dispatch is intended.",
        "type": "string"
      }
    ]
  },
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
    "hash": "9afcfd2154b057f2341b6d141659ca9a4f187945c28c222db219a8cf932295e4"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "5SeX+V8/4kLd83kmQVlzNVmmxAXqrlR5T9k+yzPNc/UPtU9OElQXZvinJeQOROnP8xW8rHvw6AKxlIS2aYQOAw==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-09-12T08:01:11.293Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Batch Cycle

Take a **set** of issues to **one** pull request. Each issue is implemented independently, in parallel; the repo's expensive verification runs **once** over the combined result.

## Autonomy contract — run to completion

This skill is **autonomous**. Once the issue set and the dispatch profile are resolved, run
every step through to a shipped PR without returning to the operator.

**Never ask the operator to approve the composed batch.** Screening and composition are this
skill's work, not a proposal for review. Emitting a batch plan and asking "approve this batch
for preparation and dispatch?" is a defect, not caution. It turns a one-command workflow into a
multi-turn negotiation, and it strands the run indefinitely whenever the operator is not
watching the terminal. Report what was dropped and why in the closing summary, **after** the
batch is enqueued — never as a gate before it.

The same holds at every later boundary. Do not stop to confirm before dispatching members,
before running the expensive gate, before repairing what the gate breaks, or before opening the
PR.

There are exactly **three** places this skill may stop, each defined elsewhere in this document:

1. The single dispatch-profile clarification question — only when no family is stated *and*
   runtime evidence matches more than one configured profile.
2. A true dependency cycle, which `batch-issues-preparation` surfaces and stops on by design.
3. Zero issues survive readiness screening, leaving nothing to dispatch.

Any other stop is unauthorised. A screening call the model feels unsure about is resolved by
**dropping the issue and saying so in the summary**, not by asking.

### Offering a menu is still asking

A multiple-choice question is the same defect wearing a different hat. Presenting the operator
with options — "keep one PR / proceed with two PRs / force it in" — and waiting for a pick is
a stop, regardless of how well-reasoned the options are or whether one is marked recommended.
If the skill can rank the options well enough to recommend one, it can take that one and say
so afterwards.

### Composition conflicts resolve downward, silently

An issue can be **ready** by Step 2 and still be a poor batch member — it needs a review path
the batch cannot give it (browser verification, a manual QA pass), it mutates data, or it is a
slice of a designed sequence already represented. This is a composition conflict, and it has a
fixed resolution: **leave the issue out of the batch and out of this run entirely.**

Do not demote it to a standalone `mode: full` entry as a consolation. That silently converts
a one-PR workflow into N PRs, pays the expensive gate an extra time, and — until
`ai-dossier#713` lands — drops the run's dispatch profile so the issue executes on the wrong
provider. One PR is the contract the operator invoked; a second PR is not a smaller version of
it.

Record the excluded issue and its conflict in the closing summary, next to the screening
drops. The operator can batch it in the next run or take it through `full-cycle-issue-skill`
deliberately. Never ask which of these they would prefer.

### Once enqueued, do not recompose

After `sched enqueue` returns, the batch is sealed and members may already be dispatched.
Do not propose removing a member, adding one, or re-splitting the set. If the composition was
wrong, say so in the summary and let the run finish — a member abandoned after dispatch leaves
a live agent running outside the scheduler, an `in-progress` claim with no owner, and a
worktree nobody will clean up (`ai-dossier#675`).

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

## Dispatch profile

Resolve the agent family before handing the issue set to preparation. Run
`ai-dossier sched status --json` in the target repository and read
`dispatch.profiles` for the configured profile names and tier commands.

- If the request states a family, map it to exactly one configured profile. An exact profile name wins; otherwise use an unambiguous provider/model alias (for example, `glm` maps to the configured `zai` family when that is the only match). Never invent a profile name.
- If the request does not state a family, use runtime evidence only when it identifies exactly one configured profile. Claude Code evidence is `CLAUDECODE`; an opencode session is matched against profiles whose tiers spawn `opencode`.
- If no family is stated and runtime evidence is absent or matches more than one profile, ask exactly once: `Which dispatch profile should this batch use? Available: <sorted names>`. Do not start preparation until the answer is supplied.
- Pass the selected key as `dispatch_profile=<name>` to `imboard-ai/git/batch-issues-preparation`. It must carry that value to every slot member and to `ai-dossier sched enqueue --dispatch <name>`.
- If preparation still returns the scheduler's inconclusive-detection refusal, do not relay the raw refusal. Ask the one profile question above, retry once with the answer, and stop with the profile names if the second attempt fails.
- If `dispatch.profiles` is empty, omit `dispatch_profile` and preserve the legacy default dispatch behavior.

Report the selected profile and whether it came from the request, runtime evidence,
or the one clarification question.

### Required handoff

1. Resolve the issue set from the request and resolve `dispatch_profile` before starting preparation.
2. Run `ai-dossier run imboard-ai/git/batch-issues-preparation --pull` and pass it the `issues` input plus the selected `dispatch_profile` input. Do not start member agents directly from this skill.
3. When a profile is selected, preparation must add `dispatch: <profile>` to every slot manifest entry and execute this exact scheduler command:

   ```bash
   ai-dossier sched enqueue --from-manifest <manifest-path> --dispatch <profile>
   ```

   Passing `dispatch_profile=<profile>` to preparation is the workflow input; it is not a substitute for the scheduler's explicit `--dispatch <profile>` flag.
4. When no profiles are configured, omit both the `dispatch_profile` input and the `--dispatch` flag so the scheduler keeps its legacy default dispatch.

## Steps

### 1. Preflight the repo — before dispatching anything

**Check the capability manifest.** `ai-dossier cap list` in the target repo. A repo with no `.dossier/automation/manifest.yaml` reports no capabilities, and undeclared gate capabilities are *skipped* — so members run with **no gate at all**, silently. Tell the operator before dispatching, not after.

**Check what the expensive gate actually costs here.** If the repo's full verification is a few minutes, batching buys little; say so and suggest fleet-cycle.

### 2. Screen the set for readiness

Cheap checks that prevent expensive failures. For each issue, read the **body**, not the labels:

- **Does every artifact it names exist on the base branch?** An issue saying "migrate onto the hook extracted by #N" depends on #N — whether or not it says "Depends on". Verify the symbol exists; do not trust the prose.
- **Does the body enumerate a countable work list?** Count it — to confirm the scope is *bounded*, not to reject it for being large. A "documentation" issue can be thousands of lines with no stated end; an issue naming 25 call sites to migrate is bounded work and belongs in the batch. Size is not a screening criterion here, and Step 3 says why.
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
