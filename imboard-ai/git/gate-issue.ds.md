---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "gate-issue",
  "title": "Gate Issue — Pre-Flight Safety Check",
  "version": "1.1.1",
  "protocol_version": "1.0",
  "status": "Stable",
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
    "hash": "e0ecac7b84f6ad1f94896002c45e9467d042ce0eece2edea75ac7bc51eb63fd9"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "jO2/kPZ5e+l/CoRGr8PWcWWH9XdF8M+RRuPZMPwkoBLxgK/D/2DQVe/BUriL14rCPdfJaaSIstsGkSJV1eyHCg==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-08-23T13:51:19.875Z",
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
- You are in a git repository with GitHub as a remote

## Actions to Perform

### Step 1: Fetch Issue Metadata

```bash
gh issue view <issue_number> --json state,labels,body
```

### Step 1b: Mint Run ID

Mint the runstate `run_id` ONCE here. Every later phase of this issue reuses it unchanged.

```bash
RUN_ID="r-<issue_number>-$(openssl rand -hex 2)"
# fallback if openssl is unavailable:
# RUN_ID="r-<issue_number>-$(date +%s | tail -c 5)"
echo "Run ID: $RUN_ID"
```

Pass `run_id` to every subsequent sub-dossier exactly like `base_branch`.

### Step 2: Hard Blocks

If ANY of these are true, abort immediately with a comment on the issue:

- **Issue is closed**: `state == "CLOSED"`
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

Then post the blocked runstate milestone (Step 6) with `status=blocked` and `reason=closed|decomposed|needs-clarification|epic|open-dependency-<N>`.

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

Post the phase milestone to the issue. This is the last step of the phase — on a hard block, post it with `status=blocked` and a `reason=` before stopping. Comments are append-only: never edit or delete a prior milestone. Do not skip this in nested or fleet mode — it is the only state that survives the session.

```bash
gh issue comment <issue_number> --body "$(cat <<EOF
<!-- runstate:v1 -->
phase=gate status=done run=<run_id> at=$(date -u +%Y-%m-%dT%H:%M:%SZ)
base_branch=<BASE_BRANCH>
warnings=<n>
next=setup
EOF
)"
```

`at` is filled in by the template (the heredoc is unquoted so `$(date …)` expands); put no other `$` in values. Values contain no spaces (use `-` or `,`); paths are absolute. On a hard block use `status=blocked`, `reason=<short-slug>`, and `next=done`.

## Output

- `status`: pass | fail
- `run_id`: runstate run id minted in Step 1b — pass to every subsequent sub-dossier
- `base_branch`: resolved base branch (default: `main`)
- `warnings`: list of soft warnings (may be empty)
- `failure_reason`: reason for hard block (only if status=fail)
- Posts runstate milestone to the issue (`phase=gate`)

## Validation

- [ ] Issue metadata was fetched
- [ ] A `run_id` was minted and reported
- [ ] All hard blocks were checked
- [ ] Soft warnings were counted
- [ ] Base branch was extracted from issue body
- [ ] On hard block: comment was posted and workflow stopped
- [ ] On pass: output includes status, base_branch, run_id, and warnings
- [ ] Runstate milestone comment was posted to the issue

## Troubleshooting

**`gh` not found**: Install GitHub CLI: https://cli.github.com/

**Issue not found**: Verify the issue number and repository access

**Dependencies check slow**: Only checks issues explicitly referenced with "Depends on #N"
