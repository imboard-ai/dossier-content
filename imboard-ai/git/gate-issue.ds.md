---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "gate-issue",
  "title": "Gate Issue — Pre-Flight Safety Check",
  "version": "1.4.0",
  "protocol_version": "1.0",
  "status": "Stable",
  "last_updated": "2026-08-24",
  "objective": "Lightweight safety gate that checks issue metadata for hard blocks and soft warnings before starting any workflow",
  "category": [
    "development"
  ],
  "tags": [
    "issue",
    "workflow",
    "git",
    "github",
    "gate"
  ],
  "risk_level": "low",
  "risk_factors": [
    "network_access"
  ],
  "requires_approval": false,
  "inputs": {
    "required": [
      {
        "name": "issue_number",
        "description": "GitHub issue number to check",
        "type": "number"
      }
    ],
    "optional": []
  },
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "content_scope": "references-external",
  "external_references": [
    {
      "url": "https://cli.github.com/",
      "description": "GitHub CLI documentation",
      "type": "documentation",
      "trust_level": "trusted",
      "required": false
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "d8c08244bec8ee77f63f0af5ebb0c983d56d271a3ad73f7fcdb78d877eb7471a"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "YtE+3+sjuwGT5mGl9fNHou+gmcS4BTYP/gw/FZynLFV2DrEzCDmNFwR4FkZClg3HIqbYXe5vvqcI6gsXFMxZCA==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-08-24T09:01:52.918Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Gate Issue — Pre-Flight Safety Check

## Objective

Lightweight safety gate (~30s) — check issue metadata for blockers before committing to a workflow. No codebase exploration.

## Prerequisites

- GitHub CLI (gh) is installed and authenticated
- `ai-dossier` CLI >= 0.10.0 is installed (`ai-dossier runstate --help`) — this phase mints, reads, and posts runstate through it
- You are in a git repository with GitHub as a remote

## Actions to Perform

### Step 1: Fetch Issue Metadata

```bash
gh issue view <issue_number> --json state,labels,body
```

### Step 1b: Mint Run ID

Mint the runstate `run_id` ONCE here. Every later phase of this issue reuses it unchanged.

```bash
RUN_ID=$(ai-dossier runstate mint --issue <issue_number>)
echo "Run ID: $RUN_ID"
```

Pass `run_id` to every subsequent sub-dossier exactly like `base_branch`.

(On a resumed run, Step 1.5 below reuses the prior `run_id` instead of this fresh mint — continuity of the trail matters more than a new id.)

### Step 1.5: Resume Detection

Check whether this issue already has runstate history before treating it as a fresh run:

```bash
LAST=$(ai-dossier runstate last --issue <issue_number> --json)
```

If there is no milestone (empty output): `resume_from=none`. Keep the `RUN_ID` minted in Step 1b and continue to Step 2 (Hard Blocks) as today.

Otherwise read `.phase`, `.status`, `.run`, and the phase-specific keys from that JSON. **Reuse `.run` as `RUN_ID`** — discard the one minted in Step 1b. Then VERIFY the claim against reality — never trust the comment alone:

**Run the verification, do not hand-walk it.** `ai-dossier runstate verify --issue <issue_number> --json` performs the entire table below and prints `resume_from` plus the parsed `resume_context` — that is the supported way to resume. Use its answer.

```bash
ai-dossier runstate verify --issue <issue_number> --json
```

Resume verification is **remote-first**: origin/<branch> is the durable copy of the work (see full-cycle-issue's WIP sync rule), so every check below verifies against the remote, not the local worktree.

*verify implements this table; run the command, read the table only when debugging.*

| last milestone | verify | if verified → | if not → |
|---|---|---|---|
| `gate` done | nothing to verify | `resume_from=setup` | — |
| `setup` done | `git ls-remote --exit-code origin <branch>` (remote is authoritative; local worktree existing is a bonus, not a requirement) | `resume_from=plan` | `resume_from=setup` |
| `plan` done | `git fetch origin <branch>`, then remote branch exists AND `head` is reachable from it: `git ls-remote origin <branch>` sha == milestone `head`, OR (if not an exact match) `git merge-base --is-ancestor <head> FETCH_HEAD` succeeds | `resume_from=implement` | `resume_from=plan` (or setup if those fail) |
| `implement` done | same remote sha/ancestor check as `plan` | `resume_from=review` | `resume_from=implement` |
| `review` done | same remote sha/ancestor check as `plan` | `resume_from=ship` | `resume_from=review` |
| `review` partial | same remote sha/ancestor check as `plan` | `resume_from=review` with `agents_pending` passed through | `resume_from=review` |
| `ship` awaiting-merge | `gh pr view <pr> --json state,mergedAt,mergeable` | merged → `resume_from=ship-teardown`; open+MERGEABLE → `resume_from=ship-wait`; CONFLICTING/closed-unmerged → `resume_from=ship` | `resume_from=ship` |
| `ship` done | — | `resume_from=report` | — |
| `report` done | issue is CLOSED? | print "already complete" and STOP (status pass, `resume_from=done`) | `resume_from=report` |
| any `blocked` | treat as the phase itself: `resume_from=<that phase>` | | |

The planning-file check (`test -f <planning>`) is no longer part of this table — it happens AFTER the worktree is materialized, in full-cycle-issue's "## Resuming" section, since the worktree may not exist on this machine yet.

Additionally, when the last milestone carried a `worktree=` path, check `test -d <worktree>` on this machine and report `local_worktree=present|absent` in the output. This is informational only — it never changes the `resume_from` decision above, since the remote check is authoritative.

**Loop cap**: if the last THREE runstate milestones are all `status=blocked` on the same phase, hard block with `reason=resume-loop` — add label `decision-pending`, post the abort comment (Step 2), and stop.

**Hard blocks (Step 2) still apply on resume** — EXCEPT "issue is closed", which is not a block when `resume_from` is `report` or `done` (a merged PR auto-closes the issue before report runs).

Carry `resume_context` forward: the parsed key=value lines of the last milestone, for later phases to consume.

### Step 2: Hard Blocks

If ANY of these are true, abort immediately with a comment on the issue:

- **Issue is closed**: `state == "CLOSED"` (not a block when resuming with `resume_from=report` or `resume_from=done` — see Step 1.5)
- **Has label `decomposed`**: Issue was already decomposed into sub-issues by triage
- **Has label `needs-clarification`**: Issue was triaged as not ready
- **Has label `epic`**: Issue is an epic, not directly implementable
- **Has open dependency**: Body contains `Depends on #N` (case-insensitive) where issue #N is still open:
  ```bash
  # For each "Depends on #N" found in the body:
  gh issue view <N> --json state --jq '.state'
  ```
  If the referenced issue state is `OPEN`, it is a hard block.

**On hard block**: Post a comment and stop:

```bash
gh issue comment <issue_number> --body "**Workflow aborted**: <reason>. Resolve the blocker and re-run."
```

Then post the blocked runstate milestone (Step 6) with `--status blocked --kv reason=closed|decomposed|needs-clarification|epic|open-dependency-<N>`.

Do NOT proceed.

### Step 3: Soft Warnings

Count how many of these are true:

- Body is empty or less than 50 characters
- Body contains research keywords: `evaluate`, `research`, `explore`, `investigate`, `compare` (case-insensitive)
- Issue has no labels at all

If **2 or more** soft warnings: log a warning line but continue:

```
⚠ Gate: 2 soft warnings (short body, no labels) — proceeding with caution
```

If 0-1 soft warnings: proceed silently.

### Step 4: Extract Base Branch

Parse the issue body for `merges into \`<branch-name>\`` (case-insensitive):

```bash
BASE_BRANCH=$(gh issue view <issue_number> --json body --jq '.body' | grep -oiP 'merges into `\K[^`]+' | head -1)
if [ -z "$BASE_BRANCH" ]; then
  BASE_BRANCH="main"
fi
echo "Base branch: $BASE_BRANCH"
```

### Step 5: Output

Report results:

```
Gate passed: #<issue_number> — <title>
Base branch: <BASE_BRANCH>
Warnings: <count> (<list or "none">)
```

### Step 6: Runstate Milestone

Post the phase milestone to the issue. This is the last step of the phase — on a hard block, post it with `--status blocked --kv reason=<short-slug>` before stopping. Comments are append-only: never edit or delete a prior milestone. Do not skip this in nested or fleet mode — it is the only state that survives the session.

```bash
ai-dossier runstate post --issue <issue_number> --phase gate --status done --run "$RUN_ID" \
  --kv base_branch=<BASE_BRANCH> \
  --kv warnings=<n> \
  --kv resumed_from=<phase|none> \
  --kv prior_run=<run_id|none> \
  --kv verified=<comma list of checks passed|none>
```

The CLI stamps `at=` and computes `next=` (here `setup`) — do not pass either. It validates phase, status, and keys and refuses a malformed milestone; never hand-write the comment instead. Values contain no spaces (use `-` or `,`); paths are absolute. On a hard block: `--status blocked --kv reason=<short-slug>` (the CLI then sets `next=done`).

## Output

- `status`: pass | fail
- `run_id`: runstate run id — minted fresh in Step 1b, or reused from a prior run on resume (Step 1.5) — pass to every subsequent sub-dossier
- `base_branch`: resolved base branch (default: `main`)
- `warnings`: list of soft warnings (may be empty)
- `failure_reason`: reason for hard block (only if status=fail)
- `resume_from`: phase to resume from, or `none` for a fresh run
- `resume_context`: parsed key=value lines of the last runstate milestone, for later phases to consume
- `local_worktree`: `present` | `absent` | `n/a` — whether the `worktree=` path from resume_context exists on this machine (informational; remote is authoritative)
- Posts runstate milestone to the issue (`phase=gate`)

## Validation

- [ ] Issue metadata was fetched
- [ ] Resume detection (Step 1.5) ran before hard blocks, via `ai-dossier runstate last` + `ai-dossier runstate verify`
- [ ] On resume: the prior `run_id` was reused, not re-minted, and the claimed milestone was verified against reality (not trusted blindly)
- [ ] Resume verification is remote-first — `setup`/`plan`/`implement`/`review` milestones are verified against `origin/<branch>` (ls-remote / merge-base ancestor check), not a local worktree
- [ ] `local_worktree=present|absent` was reported when a `worktree=` path existed in `resume_context` (bonus signal only, never gates the resume decision)
- [ ] Resume loop cap enforced — 3 consecutive `blocked` on the same phase is a hard block with `reason=resume-loop`
- [ ] A `run_id` was minted via `ai-dossier runstate mint` (fresh run) or reused (resumed run) and reported
- [ ] All hard blocks were checked (with the resume exception for a closed issue at `resume_from=report`/`done`)
- [ ] Soft warnings were counted
- [ ] Base branch was extracted from issue body
- [ ] On hard block: comment was posted and workflow stopped
- [ ] On pass: output includes status, base_branch, run_id, warnings, resume_from, and resume_context
- [ ] Runstate milestone was posted via `ai-dossier runstate post`, including `resumed_from`/`prior_run`/`verified`

## Troubleshooting

**`gh` not found**: Install GitHub CLI: https://cli.github.com/

**`ai-dossier runstate` not found**: the CLI is older than 0.10.0 — install it (`npm i -g @ai-dossier/cli`). Do not fall back to hand-written milestone comments.

**Issue not found**: Verify the issue number and repository access

**Dependencies check slow**: Only checks issues explicitly referenced with "Depends on #N"
