---dossier
{
  "dossier_schema_version": "1.0.0",
  "protocol_version": "1.0",
  "name": "batch-cycle-skill",
  "title": "Batch Cycle",
  "version": "1.6.0",
  "status": "Draft",
  "last_updated": "2026-09-24",
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
    "hash": "6c9c0d4d0d67f80cc126d7725b5e20e392c2b032a9ded892abd9a485cbce0661"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "GW/FqJY2bzEuc9x3r1OBSrOfrGDDSzAo9L+QvO7bNachbC46iqU+EzFrXufgEfg7EkgvZJvwReBG/KftdI5hAg==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-09-24T16:01:52.728Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Batch Cycle

Take a **set** of issues to **one** pull request. Each issue is implemented independently, in parallel; the repo's expensive verification runs **once** over the combined result.

**First step on the issue set, always: `ai-dossier batch compose`** (Step 2, right after the repo preflight) — a free, deterministic preview of what can actually be batched, before any model is paid for.

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
3. Zero issues survive admission and readiness screening — even after backfill — leaving nothing
   to dispatch.

**A sub-minimum batch is not formed.** Fewer than 2 members after backfill is not a batch: a
one-member batch pays the prep overhead and amortizes nothing (history before this rule: mean 2.2
members, three single-member batches). Do not form it and do not ask — hand the lone survivor to
`full-cycle-issue-skill` (still exactly one PR, the contract the operator invoked) and say so in
the summary. That is a hand-off, not a stop.

Any other stop is unauthorised. A screening call the model feels unsure about is resolved by
**dropping the issue and saying so in the summary**, not by asking.

### Offering a menu is still asking

A multiple-choice question is the same defect wearing a different hat. Presenting the operator
with options — "keep one PR / proceed with two PRs / force it in" — and waiting for a pick is
a stop, regardless of how well-reasoned the options are or whether one is marked recommended.
If the skill can rank the options well enough to recommend one, it can take that one and say
so afterwards.

### Composition conflicts resolve downward, silently — and downward includes backfill

An issue can be **ready** by Step 3 and still be a poor batch member — it needs a review path
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

**A risk-floor issue is not a composition conflict** (#770, operator decision Option A; #818).
RFC-0001 E.2 rules 1, 4 and 5 — a risk-floor area (auth, billing, security, migration), a
deploy-pipeline change, and a change predicting > 8 files — make an issue a `review=full`
member — strong tier, full-cycle-grade review in member-cycle, a risk-floor review of its
commits in batch-integrate — not an exclusion. At most **2** `review=full` members per batch
(blast radius); a third waits for the next batch. The hard exclusions are only: production data
mutation or ops, a slice of a designed sequence, decisions/epics/trackers, a different base. A
classifier `mode=full` (any other E.2 floor — visual/browser review included, since the batch
gate has no browser) hands the issue to full-cycle.

**When drops leave the batch below `min_members` (default 3), backfill — silently.** Take the
next candidates from `batch compose`'s ranked `backfill[]`, screen each one's body exactly like
a pick (Step 3 — compose admits unready features/trackers today, ai-dossier#802), respect the
`review=full` cap, and continue. Do not ask the operator whether to backfill, or which candidate
to take; name the backfilled issues in the summary. Only if backfill runs dry below 2 members
is no batch formed (above).

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
| `ai-dossier batch compose` | **free, first**: prescreen:v4 + readiness over the picks (or the backlog), returns the admissible composition, each member's `review`, and ranked backfill — zero model calls |
| `imboard-ai/git/batch-issues-preparation` | re-runs compose, screens bodies, backfills, classifies ONLY admitted members, composes, enqueues with per-member `review` |
| `ai-dossier sched` | creates the integration branch, dispatches members in parallel worktrees |
| `imboard-ai/git/member-cycle` | one agent per issue: implement, test by relevance, review at its `review` level (never zero agents), hand over |
| `imboard-ai/git/batch-integrate` | the parent: merge, verify once via `gate.batch`, repair, risk-floor review of `review=full` members, ship one PR |

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

1. Resolve the issue set from the request and resolve `dispatch_profile` before starting preparation. Run Step 2's `ai-dossier batch compose` on it first.
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

**Check for a `gate.batch` capability.** The batch gate should be the repo's CI-parity gate, paid once for the union of the members' diffs — not an unsharded full suite. `sched enqueue` refuses to form a batch in a repo whose only full gate declares itself timeout-prone and has no `gate.batch` (ai-dossier#777); if `cap list` shows that shape, say so before dispatching anything.

### 2. Compose first — `ai-dossier batch compose` (free, before any model spend)

From the target repo, before any classifier or member runs:

```bash
ai-dossier batch compose --issues <operator picks> --json      # operator named issues
ai-dossier batch compose --backlog --json                      # "batch something from the backlog"
```

**Operator picks go through compose like everything else** — hand-picked is not admitted. On the #770 evidence, five hand-picked issues collapsed to a one-member batch only after ~425k decision-grade classifier tokens; compose would have shown the outcome for free. Read its `status`:

- `ok` → proceed to preparation with the picks; compose's backfill is the fallback for later drops.
- `under-min` / `no-batch` → the picks alone cannot make a batch. Preparation will backfill from compose's ranked `backfill[]` (silently — see the autonomy contract); if backfill runs dry below 2, no batch is formed and the survivor goes to `full-cycle-issue-skill`.

Report compose's `excluded[]` (with codes) in the closing summary. Never dispatch a model for an excluded issue.

### 3. Screen the admitted members for readiness

Compose's admissibility is not readiness — it admits AC-less features and trackers as backfill today (ai-dossier#802). Cheap checks that prevent expensive failures. For each admitted member — **picks and every backfill candidate** — read the **body**, not the labels:

- **Does every artifact it names exist on the base branch?** An issue saying "migrate onto the hook extracted by #N" depends on #N — whether or not it says "Depends on". Verify the symbol exists; do not trust the prose.
- **Does the body enumerate a countable work list?** Count it — to confirm the scope is *bounded*, not to reject it for being large. A "documentation" issue can be thousands of lines with no stated end; an issue naming 25 call sites to migrate is bounded work and belongs in the batch. Size is not a screening criterion here, and Step 4 says why.
- **Is it assigned or in progress?** Someone may already be on it.
- **Is it a tracker or a decision?** A body listing many independent findings, or headed "Decision needed", has no stopping point for an agent.
- **Does it have acceptance criteria?** A feature or initiative with no AC has no stopping point.
- **Does it mutate or delete production data, or do production ops** (secret/SSM writes, DNS, third-party console config)? Those need independent revert granularity — keep them out of a shared PR.

Drop what fails, backfill the gap from compose's ranked list (screening each candidate the same way), and say why. A dropped issue costs nothing; a member forcing work against a missing dependency costs an agent run.

### 4. Compose the batch

**Prefer a mixed cohort.** Batch value is a function of the **union of members' affected scopes**, not the member count: once one member triggers the repo's expensive stage, every member added after it rides along at almost no extra gate cost. A batch of issues that all avoid the expensive stage amortizes almost nothing.

**Keep slices of one designed sequence out of the same batch** — PR1/PR2/PR3 of a feature are not independent and will conflict by construction. Either batch one of them, or expect to resolve the collision.

**Do not size the batch by predicted diff.** Diff size predicts neither cost nor conflict: measured members have run 92 turns for a net −29 lines, and 59 turns for +193.

**Risk-floor, deploy-pipeline and > 8-file members (E.2 rules 1, 4, 5) ride as `review=full`, at most 2 per batch** — see the autonomy contract. The manifest carries `review` on every member.

### 5. Dispatch and integrate

The scheduler creates the integration branch and dispatches one `member-cycle` agent per issue, in parallel, each in its own worktree — `review=full` members at strong tier minimum with full-cycle-grade review. When all members have landed or handed back, run `batch-integrate`; it refuses to ship a member whose review milestone shows no agents ran.

### 6. Ship

One PR, **rebase-merged, never squashed** — per-issue commits carry the attribution the model depends on.

### Manual recovery

If the batch leaves this skill's path — the scheduler blocks it, an agent or operator takes the
integration branch over by hand, or the PR is opened by hand — follow
`imboard-ai/git/batch-integrate` Step 6b: the PR body carries `Closes #<member>` for every
shipped member and `Refs #<anchor>` only, **never** `Closes #<anchor>`; always run Step 6a after
the merge. Close the anchor only on positive evidence — every member closed as completed by
shipped code, none evicted, handed back, or requeued — otherwise leave it open for the operator.

## What to tell the operator

- `batch compose`'s verdict: admitted, excluded (with codes), and which members were backfilled
- Which issues were dropped in screening, and why
- Which members ran as `review=full`, and why (the risk-floor reasons)
- If no batch was formed (fewer than 2 survivors): which issue was handed to full-cycle
- Any member that handed back rather than implementing — this is a valued outcome, not a failure
- Every batch-level repair the parent made, and its cause
- If the gate failed on infrastructure rather than code: what was retried, and what remains unverified

## Notes

- A batch that outlives another batch's merge inherits its changes; the base can move several commits during one batch's lifetime on an active repo. The parent merges the base again before shipping.
- Batch verification often depends on a **shared** resource (a test-database pool, a runner). It may be shared across machines, not just processes. Concurrent batches can starve each other, and the symptom looks exactly like a member regression. `batch-integrate` handles this via its four-way verdict — but if you can, do not run two batch gates at once.
