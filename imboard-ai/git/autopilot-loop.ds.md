---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "autopilot-loop",
  "title": "Autopilot Loop — Unattended, Budget-Gated Backlog Orchestration",
  "version": "1.0.0",
  "protocol_version": "1.0",
  "status": "Draft",
  "last_updated": "2026-09-29",
  "objective": "Progress a repository's GitHub backlog unattended for hours: each self-paced cycle checks a usage budget, picks the highest-value unit of work, dispatches cheaper worker agents that take it to a green-CI PR, reviews and merges it as the orchestrator, verifies releases, files follow-ups, and logs every cycle to a pinned tracking issue that doubles as the owner's steering channel",
  "category": [
    "development"
  ],
  "tags": [
    "github",
    "autonomous",
    "orchestration",
    "loop",
    "autopilot",
    "budget",
    "observability",
    "review",
    "full-cycle"
  ],
  "risk_level": "high",
  "risk_factors": [
    "modifies_files",
    "network_access",
    "executes_external_code"
  ],
  "requires_approval": false,
  "destructive_operations": [
    "Dispatches background agents that create branches, worktrees and pull requests",
    "Merges pull requests into the default branch (when merge authority is granted)",
    "Publishes releases and registry artifacts (when release authority is granted)",
    "Closes, labels and comments on GitHub issues"
  ],
  "inputs": {
    "required": [
      {
        "name": "repo",
        "description": "GitHub repository to work on, as owner/name. The local checkout must be the current working directory (or its parent for nested layouts).",
        "type": "string",
        "example": "imboard-ai/ai-dossier"
      }
    ],
    "optional": [
      {
        "name": "budget_cap_percent",
        "description": "Stop the loop when the weekly (7-day) model usage reaches this percentage. The meter is usually account-wide, so other sessions count too.",
        "type": "number",
        "default": 40
      },
      {
        "name": "short_window_pause_percent",
        "description": "When the short (e.g. 5-hour) usage window reaches this percentage, wait for its reset instead of stopping.",
        "type": "number",
        "default": 85
      },
      {
        "name": "usage_command",
        "description": "Shell command that prints current usage percentages (weekly and short window). If absent or failing, the loop must NOT dispatch new work until a reading succeeds.",
        "type": "string",
        "example": "python3 ~/projects/general/ai-usage/ai-usage.py --no-cache"
      },
      {
        "name": "authority",
        "description": "What the loop may do without asking: 'pr' (open PRs only), 'merge' (merge after green CI + orchestrator review), 'release' (merge and let releases publish). The owner sets this at kickoff.",
        "type": "string",
        "default": "merge"
      },
      {
        "name": "worker_model",
        "description": "Model for worker subagents that implement issues. The orchestrator itself should be the strongest available model; workers can be a cheaper tier.",
        "type": "string",
        "default": "sonnet"
      },
      {
        "name": "max_parallel_workers",
        "description": "Concurrent worker units per cycle. Review time, not tokens, is the real bottleneck: 2 is the sustainable default; raise only for trivially reviewable work.",
        "type": "number",
        "default": 2
      },
      {
        "name": "lessons_interval_hours",
        "description": "Run an independent lessons-learned review every N hours of loop time.",
        "type": "number",
        "default": 6
      },
      {
        "name": "log_issue",
        "description": "Existing issue number to use as the loop's log/steering channel. If omitted, the loop creates and pins one.",
        "type": "number"
      },
      {
        "name": "owner_action_label",
        "description": "Label for issues blocked on something only the owner can do (accounts, secrets, purchases, external settings).",
        "type": "string",
        "default": "user-interaction-needed"
      }
    ]
  },
  "outputs": {
    "files": []
  },
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "dbe02086d1f5cdbd140daa4414c29a4f5c795ad853d764d8066cc30f87b533d0"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "YUF9vOm3Kp6mGcf1X2DqE8IJgAdwPh8kFmoaIYrdNoo/UcSz6+lenPDOIJ7ZuYFMftk4gSF81/IeUEDNwErFBg==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-09-29T15:12:45.231Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Autopilot Loop — Unattended, Budget-Gated Backlog Orchestration

## Objective

Let an owner walk away (travel, weekend) while a repository's backlog keeps moving **with high quality and full visibility**. One **orchestrator** session (strongest model) runs a self-paced loop. Each cycle it checks the budget, picks work, dispatches **worker** subagents (cheaper model) that take issues to a green-CI PR, then reviews, merges, verifies and logs. The owner steers asynchronously through comments on a pinned log issue.

This dossier encodes a procedure that was run for 11+ cycles on a real repo (25+ merged PRs in ~6 hours), including the failures it hit. The **Rules** section is the distilled lessons. Follow them literally; each one exists because skipping it caused a real defect.

## Guiding Principles

1. **Workers produce PRs; the orchestrator merges.** A worker never merges. The orchestrator's review is the quality gate, and many environments' safety classifiers refuse unreviewed agent merges anyway.
2. **The log issue is the source of truth.** If the session dies, a new one resumes from the latest `Cycle N` comment. Anything not in the log did not happen.
3. **Escalate only what is genuinely the owner's**: spending money, public-facing content under their name, external relationships, product direction, credentials. Everything else, decide and record. Every escalation carries: the decision, alternatives, pros/cons, a recommendation, and why it could not be defaulted. Escalations never block the loop; it continues on decision-free work.
4. **Calibrate caution to reversibility.** Code merges are revertible, so ship. Data deletion, publishing irrevocable artifacts and touching other sessions' running processes are not, so be careful there.
5. **Prefer fixing the machinery the loop depends on** (release pipeline, test isolation, CI) before feature work. A broken pipeline silently wastes every later cycle.

## Prerequisites

- [ ] `gh` authenticated with push/merge rights on `repo`; an environment that can spawn background subagents and self-schedule wakeups (e.g. Claude Code `/loop` dynamic mode + `ScheduleWakeup`).
- [ ] The owner has stated: budget cap, authority level, worker model, and that the loop may run unattended. If any is missing, ask **once** at kickoff with a recommended default, then never again.
- [ ] If the environment's auto-mode classifier blocks `gh pr merge`, the owner adds a scoped allow rule (e.g. `autoMode.allow: "Bash(gh pr merge:*) for <repo> after green CI and orchestrator review"`). Never work around a denial.
- [ ] `full-cycle-issue` (or the repo's equivalent single-issue procedure) is available to workers.

## Phase 0: Kickoff (once)

1. **Read the repo's agent instructions** (AGENTS.md / CLAUDE.md / CONTRIBUTING): worktree rules, build/test commands, version-bump and release conventions. Check whether harness worktree isolation works on this repo layout; nested layouts (e.g. `main/.git` with `core.worktree`) often break it. In that case workers create worktrees themselves with **absolute paths**.
2. **Check for other actors** on the same repo: other sessions, a scheduler daemon, bots. List their open PRs and `in-progress` issues; the loop never touches them.
3. **Create and pin the log issue** (unless `log_issue` given): charter (goals, budget cap, authority, worker model, do-not-touch list), steering commands (`stop`, `skip #N`, `prioritize #N`, free text), and the cycle comment format. Label it for easy filtering.
4. **Create the owner-action label** (`owner_action_label`) with a description.
5. **Save a memory note** (if the environment has persistent memory) pointing at the log issue and the charter, so a resumed session can find it.

## Phase 1: Cycle start — gates and steering

1. **Usage gate.** Run `usage_command`.
   - Weekly ≥ `budget_cap_percent` → post a final summary on the log issue, notify the owner, stop the loop.
   - Short window ≥ `short_window_pause_percent` → schedule a wakeup after its reset, do nothing else.
   - **Reading failed** (e.g. HTTP 503) → do not dispatch new work. Merging already-green PRs is fine, since it costs nearly nothing. Retry next tick.
2. **Steering.** Read log-issue comments since the last cycle (exclude the loop's own). Obey `stop` / `skip` / `prioritize`; treat free text as guidance. Also read any direction the owner gave in-session, and **record it on the log issue** so it survives the session.
3. **Health of main.** List non-green workflow runs on the default branch since the last cycle, and compare published package versions against the versions on main. Anything red or missing becomes this cycle's first item, or a filed issue. This catches failures that a later green run masks.
4. **In-flight work.** If previous workers are still running, do not start new ones beyond `max_parallel_workers`; handle their reports as they arrive.

## Phase 2: Pick the unit of work

1. List open issues. Exclude `in-progress`, `blocked`, the owner-action label, issues with open PRs, and anything claimed by another actor.
2. Rank by: (a) security/trust defects, (b) bugs in the loop's own machinery (release, CI, test isolation, the scheduler), (c) reliability/automation bugs other actors hit today, (d) observability, (e) adopter-facing product work, (f) the rest. Old roadmap epics are rarely a cycle's unit. Triage them instead: close what's shipped with evidence, rescope the rest into concrete gaps, escalate direction questions.
3. **Batch related small issues into one unit** (same files/area, one PR). Keep parallel units on **disjoint files**.
4. **Close issues that are already resolved** with evidence (commands, versions, links) instead of re-doing them. Close deliberate non-goals with the decision and what was left out on purpose.
5. **Claim** each picked issue with the `in-progress` label, and post a `Cycle N — started` comment (usage before, picked + why, skipped + why).

## Phase 3: Dispatch workers

Spawn one background worker per unit with `worker_model`. The prompt MUST contain (template, filled in):

```
You are a worker in an unattended autopilot loop for <repo> (checkout: <path>).
Resolve <issues> in ONE PR, following <single-issue procedure> INCLUDING its
review phase (spawn a review agent; for security/scoring work make it ADVERSARIAL:
"find inputs/paths that still get through"). STOP BEFORE MERGING: once CI is
green and review findings of MEDIUM+ are fixed, report back; the orchestrator
reviews and merges. Don't pause for questions; record judgment calls in the PR.

Worktree: NEVER edit/checkout in the main checkout. Create your own with an
ABSOLUTE path: git -C <main> worktree add <abs-worktrees-dir>/<branch> -b <branch> origin/<default>.
After EVERY commit verify it landed (git status clean, git log -1); never
suppress hook output. Before reporting: git status clean AND pushed.
After your PR is merged: stop. No further edits (open a new PR only if asked).
Read the registry / source packages as truth, never snapshot copies (e.g. examples/).
<issue-specific scope, acceptance, real-data verification, version-bump rules>
Constraints: other workers are editing <paths> — stay out. Check open PRs.
Final report: PR URL, CI status, per-item outcome, test evidence, review findings
+ dispositions, release on merge, follow-ups, worktree path, git-status confirmation.
```

Put the owner's constraints (don't restart shared daemons, don't publish without the recipe, etc.) in every prompt; workers don't inherit the orchestrator's context.

## Phase 4: Relay review reports (harness gap)

When a worker spawns its own review agent, **that agent's report may be delivered to the orchestrator, not to the worker.** Workers then wait forever. So:
- When a subagent report arrives addressed to you but belonging to a worker's review, **immediately relay** it to the worker, with your required-fix list (which findings must be fixed, which are optional), via the worker's message channel.
- When a worker reports "waiting on review", check whether you already hold that report.
- When a report is truncated in delivery, ask the agent to write it to a file and read the file.

## Phase 5: Orchestrator review and merge (the quality gate)

For every PR a worker reports ready:
1. **Worktree check:** in the worker's worktree, `git status` is clean and there are no unpushed commits. (A worker once reported work "done" whose commit never landed; a hook failed with its output hidden.)
2. **CI check:** the required jobs actually ran on the **final head** (not just deployment checks). A conflicting PR has no merge ref, so PR CI does not start. Rebase first.
3. **Diff review**, focused on what the worker's review may have missed:
   - Every HIGH/MEDIUM finding from the worker's review is **fixed or filed**. A review verdict of "merge" can still hide a HIGH; read the findings, not the verdict.
   - **A new trust/validation check is traced through EVERY reader of that data**, including prompt-driven readers (dossiers, scripts) and not just the code. Missing one reader is the most common security gap.
   - **Scoring, parsing and security features get ≥5 adversarial inputs** before merge (bypasses, not just happy paths).
   - Behaviour changes that could hit other actors (new refusals, schema bumps, changed defaults) are noted in the cycle log.
   - Claims about "it works on this repo" are verified on this repo (e.g. a merge-mechanism fix that doesn't fire on a repo with native auto-merge enabled).
4. **Merge** (per `authority`), clean up the worktree and branch, and confirm the issue closed.
5. **Verify releases**: the publish run succeeded, and the version is **listed** on the registry. Fresh publishes can take minutes to appear, so poll; don't conclude "not published" from a single 404. Verify downstream artifacts too (binaries, extension packages).
6. **Self-fix vs dispatch**: CI/workflow fixes of ~30 lines or less the orchestrator makes itself (branch → PR → CI → merge). State-machine, security or multi-package work goes to a worker.

## Phase 6: Cycle close

1. Usage after.
2. File follow-ups for everything deliberately left out, anything found in review but out of scope, and anything observed but unexplained (e.g. an intermittent 403). **Don't claim a root cause you haven't proven.**
3. Post `Cycle N — done` on the log issue with: usage before/after · picked + why · PRs + releases (with links) · outcome · review verdict (what the orchestrator caught) · non-green runs on main · restart-needed items · follow-ups filed · next.
4. Summarize the same in-session, with direct links.
5. Schedule the next wakeup: if workers are running, a long fallback (≈30 min), because their completion notifications are the real wake signal. Otherwise start the next cycle now.

## Phase 7: Lessons learned (every `lessons_interval_hours`)

Spawn an **independent, read-only** reviewer (strongest model) with: the log issue, all PRs merged since the loop started, issues filed and closed, releases, Actions history, and the memory note. It produces: a sourced scoreboard, what worked, what went wrong (root causes), an **audit of the riskiest merged PRs for defects the orchestrator missed** (it may use sub-reviewers; relay their reports to it), prioritized process changes, and product-direction observations. Then:
- Post it on the log issue.
- File every real defect it finds as an issue and **fix security/trust ones before any new feature work**.
- Verify its claims against the source of truth before acting. Reviewers reading stale copies produce false positives; withdraw them explicitly.
- Apply the process changes, and update the memory note.

## Owner-action items

Anything only the owner can do becomes an issue labelled `owner_action_label` with **step-by-step instructions** (URLs, exact settings, the exact `gh secret set …` command, what "done" looks like, and what the loop does once it's done). Summarize open ones in the cycle log. Never idle waiting on them.

## Rules (distilled failures — each caused a real defect)

1. Never trust a worker's "done". Check the worktree state and CI on the final head.
2. Never trust a reviewer's "merge" verdict. Read the findings.
3. Trace every new trust check through every reader, code and prompts.
4. Adversarial inputs for scoring/security features, before merge.
5. Snapshots lie; registries and source are truth.
6. Relay review reports; don't wait for the harness to deliver them.
7. After merge the worker stops; late edits become a new PR.
8. Poll registries after publish; one 404 is not a verdict.
9. Serialize release jobs (one concurrency group); idempotent plan steps.
10. A green run later on main can mask an earlier red one. Scan all runs since the last cycle.
11. Don't touch other actors' running processes (daemons, sessions). File an owner-action issue with safe steps.
12. Usage unreadable means no new dispatch.
13. Keep ≤2 parallel workers unless the work is trivially reviewable; review attention is the bottleneck.
14. Record every owner direction on the log issue the moment it's given.

## Verification checklist

- [ ] The log issue exists, is pinned, and has a `Cycle N — done` comment for every completed cycle.
- [ ] No PR was merged without: a clean pushed worktree, green required CI on the final head, and the orchestrator's review notes.
- [ ] Every release claimed in the log is visible on its registry.
- [ ] Every deliberate omission and review finding is merged, filed, or explicitly withdrawn with a reason.
- [ ] A lessons-learned review ran at each interval, and its defects were filed or fixed.
- [ ] The loop stopped (or will stop) at the budget cap with a final summary.
