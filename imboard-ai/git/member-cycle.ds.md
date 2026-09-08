---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "member-cycle",
  "title": "Member Cycle — One Issue Inside a Batch, Verified Only Where It Is Cheap",
  "version": "1.1.0",
  "protocol_version": "1.0",
  "status": "Draft",
  "last_updated": "2026-09-08",
  "objective": "Implement ONE issue in its own worktree off a shared integration branch, test it thoroughly but scoped by relevance rather than volume, and hand over to a parent orchestrator that owns all expensive verification — so N issues pay the repo's expensive gate once instead of N times",
  "category": [
    "development"
  ],
  "tags": [
    "issue",
    "workflow",
    "batch-cycles",
    "member",
    "integration-branch",
    "handover",
    "runstate"
  ],
  "risk_level": "medium",
  "risk_factors": [
    "modifies_files",
    "network_access"
  ],
  "requires_approval": false,
  "destructive_operations": [
    "Pushes commits to its own member branch. Never to the integration branch, never to the default branch; the parent orchestrator owns integration, revert and eviction."
  ],
  "inputs": {
    "required": [
      {
        "name": "issue_number",
        "description": "The issue this member implements",
        "type": "number"
      },
      {
        "name": "batch",
        "description": "Batch id this member belongs to, carried on every milestone as batch=<id>",
        "type": "string"
      },
      {
        "name": "worktree",
        "description": "Absolute path to this member's OWN worktree, branch already checked out off the integration branch, dependencies installed",
        "type": "string"
      },
      {
        "name": "integration_branch",
        "description": "The shared branch this member's branch is based on and whose parent will verify the combined result",
        "type": "string"
      }
    ],
    "optional": []
  },
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "content_scope": "self-contained",
  "checksum": {
    "algorithm": "sha256",
    "hash": "450e853bd5ab9863d2879547e4fb31e1eb8968616b656045c6cef343adc833ce"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "3fasJRNk/Jbnj2syqApJCoCFpxBjWuc2MxBx4DK1fWeDTitRne4c24w3v775OiObZokzBqU3rFOBr9v4UzFmAw==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-09-08T23:26:46.973Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Member Cycle — One Issue Inside a Batch

## Objective

Implement one issue, well, in isolation — then hand it over. A parent orchestrator merges every member onto the integration branch and runs the repo's expensive verification **once** for the whole batch. That amortization is the entire reason this workflow exists.

**The division of labour, and the one principle behind it: each level verifies what only it can see.**

- **You** can see your own change and are best placed to verify it deeply. Be thorough.
- **The parent** is the only actor that can see the *combination*. It runs the full suite, the repo's parity gate, and any browser pass — once, for everyone.

These reinforce each other. **Thorough members are what make a single expensive run viable.** If members hand over code they have not convinced themselves of, the one shared run fails constantly and the batch serialises on a parent untangling N changes at once — spending more than the runs saved.

**Non-responsibilities:** the repo-wide suite, any CI-parity/full-gate script, full e2e matrices, cross-package integration, the PR, the merge, the deploy, teardown, aggregate review, anything about sibling members. You never create a worktree or branch, never open a PR, never touch the integration branch directly.

## Prerequisites

- `gh` installed and authenticated; `ai-dossier` CLI available (beware shadow copies — a repo-local `node_modules/.bin/ai-dossier` can shadow the global install; call the newer binary by absolute path if a documented subcommand reports `unknown command`).
- You are dispatched into a prepared worktree. You do not create one.

## Actions to Perform

### Step 0: Preconditions — assert, never assume

```bash
cd "<worktree>"
git rev-parse --is-inside-work-tree     # must succeed
git branch --show-current                # must be YOUR member branch
git status --porcelain                   # must be empty
```

Any failure: post `runstate ... --status blocked --kv reason=<slug> --kv batch=<batch>` and hand back **without touching a file**. A dirty tree is the parent's call to recover, not yours.

Confirm the dependency marker a fresh clone lacks (`node_modules/`, `vendor/`, `.venv/`) is present. Absent → `reason=env-cold`; warm-up was owed to you.

Record where you started — every later diff is scoped to it:

```bash
BOUNDARY=$(git rev-parse HEAD)
```

### Step 1: Understand the issue, and check it is actually implementable

Read the issue in full, including comments. Then apply two checks that are cheap now and expensive later:

**Does every artifact the issue names actually exist on your base?** An issue that says "migrate onto the hook extracted by #N", "use the helper added in #N", or "follow the pattern from #N" is depending on #N's work. If that work is unmerged, the issue is not implementable — **regardless of how the dependency is phrased.** Verify by looking for the symbol or file on your base branch, not by trusting the prose.

**Does the body enumerate a countable work list?** "Document the five API families" that turns out to name eight is not a small issue. Count what the body actually asks for, not what its title or labels suggest.

If either check fails, **stop and hand back** (Step 6). This is a correct and valued outcome — a member that hands back in minutes costs a fraction of one that forces work against a missing dependency.

### Step 2: Implement, and be thorough

Plan, implement, write tests, run them, iterate until you are genuinely confident in your own change. Being thorough here is the point — the parent's single expensive run is the last line of defence, not the first.

**Scope your tests by RELEVANCE, not volume.** Run:

- tests you wrote for this change
- existing tests covering the files and modules you changed
- **tests for direct consumers, one hop out** — the integration test that exercises the module you edited counts, even when you did not touch that test's file
- typecheck and lint/format over your changed surface

Do **not** run: the repo-wide suite, the repo's CI-parity or full-gate script, full e2e matrices, cross-package integration, deploy. Those cost tens of minutes and run **once** at batch level. Running one yourself is the exact duplication this workflow exists to eliminate. If you believe one is genuinely needed, **say so in the handover instead of running it.**

**Lint and format your changed files explicitly**, and note the command and the paths it covered. A member's unformatted file fails a shared gate and costs the parent an entire expensive cycle to discover.

### Step 2b: Prove every test you wrote actually runs

**Writing a test is not running it.** Before you move on, for each test you added:

1. **Observe it FAIL against the unfixed code** — revert your change, or neuter the line the
   test targets, and watch the test go red for the reason you expect. Then restore and watch
   it go green.
2. If a test cannot be made to fail, it is asserting nothing. Find out why before keeping it.

This is the single cheapest defect check in this workflow, and skipping it is the most common
way a member ships a broken batch. Two measured failures, both from tests authored against an
*imagined* contract rather than an executed one:

- a test pinned the exact wording of a UI string that the code never produced
- a test asserted an emit sequence behind a request validator its fixture never satisfied, so
  the handler diverted to the error path and **neither assertion was ever reachable** — the
  test had never passed, from the moment it was written

Both looked correct in review. Both were caught by a shared gate long after the member
declared success, at the cost of an entire batch verification cycle.

Say in your handover that you did this, and what you saw. "Proved red by neutering X, restored,
green" is the sentence a parent needs.

### Step 3: Conformance — your own verdict on your own ACs

Check your change against the issue's acceptance criteria, honestly, before anyone else sees it. This is the signal that lets the parent distinguish **fix this** (tests red, ACs met) from **evict this** (ACs not met). Getting it wrong in your own favour is worse than handing back.

If the ACs cannot be met as written, say so in the handover and stop. Do not adjust the criteria to fit what you built.

### Step 4: Commit — before you claim anything is verified

**A fix that exists only in your working tree does not exist.** Commit and push to your member branch, then verify against the committed state.

```bash
git add -A && git commit -m "<type>: <subject> (#<issue_number>)"
git push -u origin "$(git branch --show-current)"
```

Never push to the integration branch or the default branch.

A useful diagnostic if a check passes locally but fails for the parent: **a stack trace citing a line number your patch moved means the running code does not contain your patch.**

### Step 5: Handover — the artifact the parent depends on

Post a comment on the issue whose first line is exactly `## handover:v1`, containing:

| Field | Why the parent needs it |
|---|---|
| Files changed, and why | orients a reader in a diff they did not write |
| **The exact test commands you ran, and their results** | tells the parent what is already covered |
| **The direct consumers you considered one hop out** | name them even where you ran none, and say why |
| The lint/format command and the paths it covered | a shared formatting gate failure is otherwise found the expensive way |
| **What you deliberately did NOT verify** | names the blast radius the parent must cover |
| Assumptions and points of uncertainty | where a failure is most likely to be genuine |
| Your conformance verdict | fix-vs-evict, per Step 3 |

Two behaviours make a handover genuinely useful, and both are worth the words:

- **If you find a pre-existing failure, prove it is pre-existing** — reproduce it with your own changes reverted, say so, and do not fix it. Without this the parent burns time attributing a failure to a member that did not cause it.
- **Offer the parent a repair option where you can see one.** "If X needs tuning, it is a one-line change at Y, and the test asserts a relation rather than a literal so tuning will not break it" turns a possible eviction into a minute of work.

If your change is visual, say plainly what a reviewer should look at and at which viewport — the parent runs one browser pass for the whole batch.

### Step 6: Handing back

If the issue is not implementable, is larger or different than it reads, or its ACs cannot be met — **stop and say so in the handover.** Post `runstate ... --status blocked --kv reason=<slug>`, leave the tree clean, and report.

Reporting this is a correct outcome and is treated as such. **A member that forces a green is worse than one that hands back.**

## Success Criteria

- Exactly one member branch, pushed, with commits scoped to this issue
- Relevance-scoped verification actually run, and named in the handover
- No repo-wide suite, parity gate, or e2e matrix executed
- `## handover:v1` posted, including what was not verified and the conformance verdict
- Integration branch and default branch untouched

## Rationale — why these clauses exist

Every rule above is here because its absence was measured in a real batch.

- **Relevance scoping, "one hop out"** — a member changed a module and never ran the pre-existing integration test that exercises it. The defect surfaced 81 minutes into the parent's gate instead of in seconds.
- **Name the lint command** — a member's four unformatted files failed a shared gate and cost a full expensive cycle. After the requirement was added, four of four members reported it and the gate passed first time.
- **Commit before claiming verified** — a repair verified against an uncommitted working tree shipped a branch that did not contain it; CI found it, at the cost of a cycle.
- **Prove a pre-existing failure** — one member did exactly this, and it is the reason the parent did not chase a failure that was not the batch's.
- **Handing back is valued** — a member found its issue depended on unmerged work, declined to copy that work forward, and handed back in four minutes. Forcing it would have cost far more and produced a divergent duplicate.
- **A member's own tests can pin a bug.** One asserted grammatically wrong copy; a repo-wide guard caught what the member's expectations encoded. Your tests are necessary, not sufficient — which is why the shared gates exist and why you must not route around them.
