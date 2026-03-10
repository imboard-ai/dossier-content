---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "issue-triage",
  "title": "Issue Triage Workflow",
  "version": "1.0.0",
  "status": "Draft",
  "last_updated": "2026-03-10",
  "objective": "Assess a GitHub issue's readiness for autonomous implementation and route it to the right lane: autonomous, plan-first decomposition, or not-ready",
  "category": [
    "development"
  ],
  "tags": [
    "github",
    "issues",
    "triage",
    "workflow",
    "autonomous",
    "planning"
  ],
  "risk_level": "low",
  "requires_approval": false,
  "risk_factors": [
    "network_access"
  ],
  "destructive_operations": [
    "Creates sub-issues on GitHub (plan-first lane)",
    "Adds labels to issues",
    "Posts comments on issues"
  ],
  "tools_required": [
    {
      "name": "gh",
      "version": ">=2.0.0",
      "check_command": "gh --version"
    }
  ],
  "relationships": {
    "followed_by": ["imboard-ai/git/full-cycle-issue"]
  },
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "5bbb9997ed83f0fadeed749372ef082e80678a95996d6b4a80008b63ca85fc7e"
  }
}
---

# Issue Triage Workflow

## Objective

Assess a GitHub issue before committing to autonomous implementation. Route it into the right lane so agents don't waste cycles on issues that need human decisions first.

## Three Lanes

| Lane | When | Outcome |
|------|------|---------|
| **Autonomous** | Clear requirements, small scope, mechanical decisions | Dispatch to `full-cycle-issue` |
| **Plan-first** | Medium complexity, some design decisions needed | Decompose into automatable sub-issues, stop |
| **Not-ready** | Vague, needs UX research, blocked dependencies | Comment with gaps + next steps, stop |

## Guiding Principle

Be conservative. It is far better to classify an issue as plan-first or not-ready than to let the full-cycle agent churn on something it can't finish. Autonomous should only be chosen when you are highly confident the issue can be implemented without human input.

## Actions to Perform

### Phase 1: Gather Issue Context

1. Extract the issue number from user input
2. Fetch the issue:
   ```bash
   gh issue view <number> --json title,body,labels,comments,milestone,state,assignees
   ```
3. **If the issue is closed**, stop immediately:
   ```
   Issue #<number> is closed. Nothing to triage.
   ```
4. **Check for existing triage comments** — search comments for "Triage:" prefix:
   ```bash
   gh issue view <number> --json comments --jq '.comments[].body' | grep -c "^Triage:"
   ```
   If already triaged and user did not explicitly request re-triage, stop:
   ```
   Issue #<number> was already triaged. Use "re-triage issue #<number>" to force re-assessment.
   ```
5. Extract signals from the issue:
   - **Action verbs in title**: create, add, fix, update, remove, refactor, migrate, etc.
   - **Body structure**: Does it have sections? Acceptance criteria? Steps to reproduce?
   - **Acceptance criteria**: Explicit checkbox items or "should" statements
   - **Dependency references**: "Depends on #N", "blocked by #N", "after #N"
   - **File paths mentioned**: Specific files/directories referenced in the body
   - **Ambiguity markers**: "or", "maybe", "could", "TBD", "decide", "explore", "research"
   - **UX/design keywords**: "mockup", "wireframe", "design", "UX", "layout", "user flow"

### Phase 2: Codebase Scope Estimation

1. **Search for related files** using keywords from the issue title and body:
   ```bash
   # Use grep/glob to find files related to the issue
   ```
   Search for: function names, component names, route paths, model names mentioned in the issue.

2. **Estimate scope**:
   - Count files that would likely need modification
   - Identify which packages are affected (frontend, backend, shared-types, etc.)
   - Check if test files exist for the affected areas
   - Classify the change type: **additive** (new files/functions), **modification** (changing existing), or **refactor** (restructuring)

3. **Check for existing patterns**: Is there similar code in the codebase that can be followed? A feature with the same shape? This affects decision autonomy — if there's a clear pattern to follow, the agent can make mechanical decisions.

### Phase 3: Score and Classify

Score the issue on four dimensions:

#### Clarity (0-3)
- **3**: Explicit acceptance criteria, steps to reproduce (for bugs), or clear feature spec
- **2**: Descriptive body with enough detail to implement, but no formal acceptance criteria
- **1**: Brief description, some context but gaps in what "done" looks like
- **0**: Title only, or body is vague/aspirational ("make it better", "improve UX")

#### Scope (0-3)
- **3**: Small — 1-3 files to modify, single package
- **2**: Medium — 4-8 files, possibly 2 packages
- **1**: Large — 9+ files, multiple packages, or significant new feature
- **0**: Unknown — can't estimate from the issue description

#### Decision Autonomy (0-3)
- **3**: Purely mechanical — bug fix with clear repro, add field to existing form, update text
- **2**: Minor decisions — naming, file placement, but existing patterns to follow
- **1**: Some product decisions — but constrained (e.g., "add a dashboard" where layout is open but components are known)
- **0**: Major product/UX decisions — multiple viable approaches, user-facing trade-offs, needs human judgment

#### Dependencies (0-2)
- **2**: No dependencies or blockers
- **1**: Soft dependencies (references other issues but not hard-blocked)
- **0**: Hard block — "Depends on #N" where #N is still open, or requires external integration not yet available

#### Classification Logic

```
IF dependencies == 0 → NOT_READY (hard block exists)
ELSE IF total >= 8 AND clarity >= 2 AND decisions >= 2 → AUTONOMOUS
ELSE IF total >= 5 AND clarity >= 1 AND decisions >= 1 → PLAN_FIRST
ELSE → NOT_READY
```

**Print the scorecard:**
```
Triage: Issue #<number>

| Dimension          | Score | Rationale |
|--------------------|-------|-----------|
| Clarity            | X/3   | <reason>  |
| Scope              | X/3   | <reason>  |
| Decision Autonomy  | X/3   | <reason>  |
| Dependencies       | X/2   | <reason>  |
| **Total**          | **X/11** |        |

Classification: <AUTONOMOUS | PLAN_FIRST | NOT_READY>
```

### Phase 4: Execute Lane

#### Lane: AUTONOMOUS

1. Post a triage comment on the issue:
   ```bash
   gh issue comment <number> --body "$(cat <<'EOF'
   Triage: **AUTONOMOUS** (score: X/11)

   | Dimension | Score | Rationale |
   |-----------|-------|-----------|
   | Clarity | X/3 | <reason> |
   | Scope | X/3 | <reason> |
   | Decision Autonomy | X/3 | <reason> |
   | Dependencies | X/2 | <reason> |

   Dispatching to full-cycle-issue workflow.
   EOF
   )"
   ```
2. Add the `triaged` label:
   ```bash
   gh label create "triaged" --color "0E8A16" --description "Issue has been triaged by agent" --force
   gh issue edit <number> --add-label "triaged"
   ```
3. **Invoke the full-cycle-issue workflow in the same session:**
   ```bash
   ai-dossier run imboard-ai/git/full-cycle-issue --pull
   ```
4. Follow ALL phases in the workflow output. Do not skip any.

#### Lane: PLAN_FIRST

1. **Explore the codebase deeper** — for each area the issue touches:
   - Read the relevant source files
   - Understand the existing patterns and interfaces
   - Identify what decisions are already made by the codebase (e.g., existing UI patterns, API conventions)

2. **Decompose into sub-issues** — each sub-issue must meet ALL of these criteria:
   - **Single concern**: One logical unit of work
   - **1-5 files scope**: Small enough for autonomous implementation
   - **Testable independently**: Can be verified without the other sub-issues
   - **No product decisions remaining**: All UX/business choices are resolved in the sub-issue description
   - **Existing pattern to follow**: References concrete code in the codebase

3. **Classify each sub-issue** into one of three categories:
   - **Ready now**: Can be autonomously implemented immediately
   - **Needs human input**: Requires a specific decision from the human (state exactly what decision)
   - **Blocked**: Cannot start until a specific "needs human input" item is resolved

4. **Determine execution order**:
   - Group sub-issues that can run in parallel
   - Identify sequential chains (where one must complete before the next)
   - Note which "ready now" sub-issues can start immediately vs. which depend on human input items

5. **Create sub-issues on GitHub** — for each decomposed sub-issue:
   ```bash
   gh issue create --title "<concise title>" --body "$(cat <<'EOF'
   Parent: #<parent-number>

   ## Description
   <clear description of what to implement>

   ## Files to Modify
   - `<file-path>` — <what changes>
   - `<file-path>` — <what changes>

   ## Acceptance Criteria
   - [ ] <criterion 1>
   - [ ] <criterion 2>

   ## Test Plan
   - <how to verify this sub-issue independently>

   ## Pattern Reference
   See `<file-path>` for the existing pattern to follow.

   ## Dependencies
   <"None" or "After #N: <reason>">

   ## Category
   <Ready now | Needs human input: <what decision> | Blocked until #N>
   EOF
   )" --label "sub-issue"
   ```

6. **Comment decomposition summary on the parent issue**:
   ```bash
   gh issue comment <parent-number> --body "$(cat <<'EOF'
   Triage: **PLAN_FIRST** (score: X/11)

   | Dimension | Score | Rationale |
   |-----------|-------|-----------|
   | Clarity | X/3 | <reason> |
   | Scope | X/3 | <reason> |
   | Decision Autonomy | X/3 | <reason> |
   | Dependencies | X/2 | <reason> |

   ## Decomposition

   ### Ready Now (can run autonomously)
   - #<sub-1> — <title> [parallel group A]
   - #<sub-2> — <title> [parallel group A]
   - #<sub-3> — <title> [after #<sub-1>]

   ### Needs Human Input
   - #<sub-4> — <title>
     Decision needed: <specific question>
   - #<sub-5> — <title>
     Decision needed: <specific question>

   ### Blocked
   - #<sub-6> — <title>
     Blocked until: #<sub-4> decision resolved

   ## Execution Order
   1. **Parallel group A**: #<sub-1>, #<sub-2> (run with `batch-issues.sh <sub-1> <sub-2>`)
   2. **Sequential after A**: #<sub-3> (run after group A merges)
   3. **After human input**: #<sub-4>, #<sub-5> (need decisions first)
   4. **After decisions**: #<sub-6> (run after #<sub-4> resolved)
   EOF
   )"
   ```

7. **Label the parent issue**:
   ```bash
   gh label create "decomposed" --color "C2E0C6" --description "Issue has been decomposed into sub-issues" --force
   gh label create "triaged" --color "0E8A16" --description "Issue has been triaged by agent" --force
   gh issue edit <parent-number> --add-label "decomposed" --add-label "triaged"
   ```

#### Lane: NOT_READY

1. **Identify what's missing** — be specific about what the human needs to provide:
   - If vague: what questions need answers
   - If blocked: which dependency must be resolved first
   - If UX decisions needed: what the specific trade-offs are
   - If research needed: what needs to be investigated and by whom

2. **Comment on the issue**:
   ```bash
   gh issue comment <number> --body "$(cat <<'EOF'
   Triage: **NOT_READY** (score: X/11)

   | Dimension | Score | Rationale |
   |-----------|-------|-----------|
   | Clarity | X/3 | <reason> |
   | Scope | X/3 | <reason> |
   | Decision Autonomy | X/3 | <reason> |
   | Dependencies | X/2 | <reason> |

   ## What's Missing

   <numbered list of specific gaps>

   ## Suggested Next Steps

   <numbered list of actions the human can take to make this issue ready>

   Once the above are addressed, re-triage this issue.
   EOF
   )"
   ```

3. **Label the issue**:
   ```bash
   gh label create "needs-clarification" --color "D93F0B" --description "Issue needs more detail before implementation" --force
   gh label create "triaged" --color "0E8A16" --description "Issue has been triaged by agent" --force
   gh issue edit <number> --add-label "needs-clarification" --add-label "triaged"
   ```

## Validation

- [ ] Issue fetched and signals extracted
- [ ] Codebase searched for related files
- [ ] All four dimensions scored with rationale
- [ ] Classification matches the scoring rules
- [ ] Correct lane executed:
  - Autonomous: full-cycle-issue invoked
  - Plan-first: sub-issues created with proper structure, parent labeled `decomposed`
  - Not-ready: gaps identified, issue labeled `needs-clarification`
- [ ] Triage comment posted on the issue

## Troubleshooting

**`gh` not found**: Install GitHub CLI: https://cli.github.com/
**Issue already triaged**: Skip unless user explicitly requests re-triage
**Score is borderline**: Prefer the more conservative lane (plan-first over autonomous, not-ready over plan-first)
**No codebase access**: Score scope as 0 and decision autonomy conservatively — this will bias toward not-ready, which is correct when you can't assess the codebase
**Sub-issue count too high**: If decomposition yields more than 7 sub-issues, the parent issue is likely too large. Consider grouping related changes or suggesting the human break it into epics first.
