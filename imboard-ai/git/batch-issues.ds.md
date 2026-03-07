---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "batch-issues",
  "title": "Batch Issue Orchestration",
  "version": "2.0.0",
  "status": "Stable",
  "last_updated": "2026-03-06",
  "objective": "Batch-process GitHub issues via Claude Code headless agents, with parallel and sequential (epic) modes",
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
  "risk_level": "medium",
  "requires_approval": false,
  "risk_factors": [
    "modifies_files",
    "network_access",
    "creates_pull_request",
    "merges_code"
  ],
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "22479a07c9df5f439caf8c1fa27c8d4006b401272a94130222ccc578b366cd40"
  }
}
---

# Batch Issue Orchestration

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
