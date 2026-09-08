---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "batch-issues",
  "title": "Batch Issue Orchestration — SUPERSEDED (pre-RFC-0001)",
  "version": "2.1.0",
  "protocol_version": "1.0",
  "status": "Deprecated",
  "last_updated": "2026-09-08",
  "objective": "SUPERSEDED. Pre-RFC-0001 batch orchestration via headless agents, predating the batch-cycles programme entirely. For batching issues into ONE PR with one verification run use imboard-ai/skills/batch-cycle-skill. For N issues to N PRs use imboard-ai/skills/fleet-cycle-skill. Kept for reference only.",
  "category": [
    "development",
    "orchestration"
  ],
  "tags": [
    "github",
    "issues",
    "batch",
    "parallel",
    "sequential",
    "epic",
    "orchestration",
    "autonomous",
    "agent-friendly"
  ],
  "risk_level": "high",
  "risk_factors": [
    "modifies_files",
    "network_access",
    "creates_pull_request",
    "merges_code"
  ],
  "requires_approval": false,
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "f8d175b07c41905b5ca0cb8d40aac8742ea789ef2216890fda1608cfb9be0365"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "+mSDR5nQZTd1TcWm2Y+7u+0Jj+n+zYrRLcP/UJ+FFmgFd6/E6ActpGZQn4qXublYMS5Sd7bSt7cCWJOob9PSBA==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-09-08T22:46:03.699Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Batch Issue Orchestration

> ## ⚠️ SUPERSEDED — this is probably not what you want
>
> This dossier predates RFC-0001 batch-cycles and orchestrates a different thing. It sorts early in a
> registry search for "batch"; it is **not** the current entry point.
>
> - **N issues → ONE PR, one expensive verification run** → `imboard-ai/skills/batch-cycle-skill`
> - **N issues → N PRs, one per issue** → `imboard-ai/skills/fleet-cycle-skill`
>
> Retained for reference; do not dispatch.


## Problem

You have a backlog of GitHub issues that can each be solved autonomously by an AI agent. You need a way to:
- Launch multiple agents **in parallel** for independent issues
- Work through an **epic label sequentially**, where each issue builds on the previous one's merged changes
- Get a clear, machine-readable summary of what succeeded, failed, or stalled

## Solution

A shell script (`scripts/batch-issues.sh`) that supports two modes:

| Mode | Flag | Concurrency | Use case |
|------|------|-------------|----------|
| **Parallel** (default) | `--max-parallel N` | Up to N agents | Independent issues |
| **Epic** | `--label LABEL` | Sequential (1 at a time) | Ordered backlog where each issue builds on the last |

Issues can be specified as individual numbers, ranges (`102..110` expands to all open issues in range), or fetched by GitHub label.

## Source Code

The canonical implementation lives in the ai-dossier repository:

> **[scripts/batch-issues.sh](https://github.com/imboard-ai/ai-dossier/blob/main/scripts/batch-issues.sh)**
>
> Raw: `https://raw.githubusercontent.com/imboard-ai/ai-dossier/main/scripts/batch-issues.sh`

To install in your project:
```bash
curl -o scripts/batch-issues.sh https://raw.githubusercontent.com/imboard-ai/ai-dossier/main/scripts/batch-issues.sh
chmod +x scripts/batch-issues.sh
```

## Prerequisites

- Claude Code CLI (`claude`) installed and authenticated
- GitHub CLI (`gh`) installed and authenticated
- `jq` installed
- A full-cycle-issue skill/workflow so agents know what to do when they receive "full cycle issue #N"

## Usage

```bash
# Three specific issues in parallel (default max 3)
./scripts/batch-issues.sh 102 103 104

# All open issues in a range
./scripts/batch-issues.sh 102..110

# Epic mode: work through a labeled backlog sequentially
./scripts/batch-issues.sh --label epic/v2

# Mix ranges, labels, and individual issues
./scripts/batch-issues.sh --label sprint-3 200 201

# High parallelism with a specific model
./scripts/batch-issues.sh --max-parallel 5 --model opus 102..120

# Dry run - preview what would execute
./scripts/batch-issues.sh --dry-run --label epic/v2

# JSON summary appended after human output
./scripts/batch-issues.sh --json 102 103

# Agent mode: stdout is pure JSON, progress on stderr
./scripts/batch-issues.sh --agent --label epic/v2
./scripts/batch-issues.sh --agent 102..110 | jq '.[] | select(.status != "merged")'
```

## Options

| Flag | Default | Description |
|------|---------|-------------|
| `--label LABEL` | | Fetch open issues by GitHub label (epic mode, forces sequential) |
| `--max-parallel N` | 3 | Max concurrent agents (ignored in epic mode) |
| `--model MODEL` | opus | Claude model to use |
| `--dry-run` | | Print commands without executing |
| `--json` | | Append structured JSON summary array to stdout |
| `--agent` | | Machine-readable mode: implies `--json`, all progress on stderr, stdout is pure JSON |

## Output

**Human output** - per-issue status with colored indicators:
- `✓ merged` - PR merged and/or issue closed
- `◐ PR open` - PR created but not yet merged
- `✗ failed` - agent exited with error or produced no output
- `? unclear` - agent exited 0 but no PR found

**JSON output** (`--json` or `--agent`):
```json
[
  {
    "issue": 102,
    "status": "merged",
    "pr": 45,
    "issue_state": "CLOSED",
    "exit_code": 0,
    "log_file": "/tmp/batch-issues/102.log",
    "log_lines": 234
  }
]
```

Status values: `merged` | `pr_open` | `failed` | `no_output` | `unclear`

Logs are written to `/tmp/batch-issues/<issue-number>.log` (one per issue).

## Key Design Decisions

**Why `nohup`?** Background processes (`&`) die when the parent terminal closes. `nohup` detaches them.

**Why pipe via stdin?** `claude -p --allowedTools "Bash,Read,..." "prompt"` fails because `--allowedTools` is variadic and consumes the prompt as another tool name. Piping via `echo "prompt" | claude -p` avoids this.

**Why check GitHub state?** Exit codes lie. An agent killed by timeout exits non-zero but may have already merged the PR. The script checks `gh issue view` and `gh pr list` for ground truth.

**Why not `set -e`?** A failed agent (non-zero `wait`) would kill the entire batch. We want to continue and report all outcomes.

**Why sequential for epics?** When issues in a label depend on each other (e.g., issue #2 needs code from issue #1), parallel execution causes merge conflicts. Epic mode forces `--max-parallel 1` so each agent starts from the latest main.

## Actions to Perform

When the user asks to set up batch issue orchestration in their project:

1. Download the script: `curl -o scripts/batch-issues.sh https://raw.githubusercontent.com/imboard-ai/ai-dossier/main/scripts/batch-issues.sh`
2. Make it executable: `chmod +x scripts/batch-issues.sh`
3. Verify prerequisites: `which claude && which gh && which jq && echo "ready"`

## Lessons Learned

These are pitfalls discovered through real batch runs:

- **`--allowedTools` is variadic** - it eats the next positional arg. Pipe the prompt via stdin instead.
- **Exit codes lie** - an agent killed by CI timeout exits non-zero but may have already merged. Always check GitHub state.
- **`set -e` kills the batch** - one failed agent aborts everything. Use `set -uo pipefail` without `-e`.
- **Agents need worktree isolation** - without it, parallel agents corrupt each other's git state. Add a `CLAUDE.md` to your repo root with worktree-only rules.
- **Pre-existing test failures cause loops** - agents waste time trying to fix tests they didn't break. The full-cycle workflow should check if failures exist on the base branch.
- **`gh pr merge --delete-branch` fails in worktrees** - it tries to checkout main locally, which conflicts. Merge with `--squash` only and clean up branches in teardown.
