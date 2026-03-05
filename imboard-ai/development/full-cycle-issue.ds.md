---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "full-cycle-issue",
  "title": "Full Cycle Issue Workflow",
  "version": "1.2.0",
  "status": "Draft",
  "last_updated": "2026-03-05",
  "objective": "Take a GitHub issue from start to merged PR autonomously — setup, implement, test, commit, push, PR, parallel review, and merge with zero unnecessary interruptions",
  "category": [
    "development"
  ],
  "tags": [
    "github",
    "issues",
    "workflow",
    "autonomous",
    "full-cycle",
    "pr",
    "merge"
  ],
  "risk_level": "medium",
  "requires_approval": false,
  "risk_factors": [
    "modifies_files",
    "network_access",
    "creates_pull_request",
    "merges_code"
  ],
  "destructive_operations": [
    "Creates new git branch",
    "Creates new git worktree",
    "Pushes branch to remote",
    "Creates and merges pull request",
    "Deletes branch after merge"
  ],
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "6841329b521e0d669bb5a071095043a61dd38888b0039df1814192a563a19c18"
  }
}
---

# Full Cycle Issue Workflow

## Objective

Take a GitHub issue from start to merged PR autonomously. For small-to-medium issues where requirements are clear.

## Guiding Principle

**Do not stop to ask yes/no questions.** Only pause when:
- The issue description is too vague to implement
- A business/product/design decision is genuinely ambiguous
- Tests fail after 2 fix attempts with unclear path forward
- Merge conflicts require human judgment

Do NOT ask about: file names, branch names, commit messages, PR descriptions, whether to proceed, or any mechanical decision.

## Prerequisites

- [ ] Git is installed and configured
- [ ] GitHub CLI (gh) is installed and authenticated
- [ ] You are in a git repository with GitHub as a remote
- [ ] You have push access

## Actions to Perform

### Phase 1: Setup

1. Extract the issue number from user input
2. **Record the original working directory** — you will return here after merge
3. Run the setup workflow:
   ```bash
   ai-dossier run imboard-ai/development/git/setup-issue-workflow
   ```
4. Provide the issue number when prompted
5. **When asked where to work, always choose option 1 (create a new git worktree)**. Do not use current directory or custom path — full-cycle must be isolated.
6. Note the worktree path from the setup output
7. `cd` into the worktree directory — **all subsequent work happens here**

### Phase 2: Understand & Plan

1. Read issue details: `gh issue view <number> --json title,labels,body,assignees`
2. Read PLANNING.md created by setup
3. Explore relevant code
4. If clear enough, proceed immediately
5. If genuinely ambiguous, ask ONE focused question then proceed

### Phase 3: Implement

1. Implement the solution following existing code patterns
2. Keep changes minimal and focused
3. Do not add tests inline — Phase 4 handles testing separately

### Phase 4: Test

1. **Detect test framework**: Look for jest.config, vitest.config, .mocharc, pytest.ini, or test scripts in package.json / pyproject.toml
2. **If tests exist for changed files** — run them. Fix failures (max 2 attempts). If still failing with unclear path, ask the user.
3. **If no tests exist for the changed code** — create focused unit tests:
   - Test the public API of changed/new modules
   - Cover happy path + key edge cases + error paths
   - Follow existing test patterns and conventions in the repo
   - Place tests where the project convention expects them (e.g., `__tests__/`, `*.test.ts`, `*.spec.ts`)
4. **Run the full test suite** to catch regressions (e.g., `npm test`)
5. Fix any regressions caused by your changes (max 2 attempts)

### Phase 5: Commit & Push

1. Stage relevant files (never `.env`, credentials, secrets)
2. Commit with conventional commits format:
   ```
   feat|fix|chore: <description>

   Closes #<issue-number>

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```
3. Push: `git push -u origin <branch-name>`

### Phase 6: Create PR

```bash
gh pr create --title "<short title>" --body "$(cat <<'EOF'
## Summary
<1-3 bullet points>

Closes #<issue-number>

## Test plan
- <how to verify>

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

### Phase 7: Parallel Review

Run 5 focused review agents **in parallel** using the Agent tool. Pass each agent the list of changed files from `git diff main...HEAD --name-only`.

Launch all 5 simultaneously:

#### Agent 1: DRY Review

> You are reviewing branch changes for DRY (Don't Repeat Yourself) violations.
> AI agents frequently rewrite code that already exists in the codebase — your job is to catch this.
>
> For each changed file:
> 1. Read the file fully
> 2. Search the **entire codebase** for existing functions, utilities, or patterns that do the same thing
> 3. Flag duplicated logic (>5 similar lines), reimplemented helpers, or missed utility reuse
> 4. Check if anything was reimplemented that's available in project dependencies
>
> For each finding: file, lines, what's duplicated, where the original lives, auto-fixable (yes if it's a straight replacement with an import).
> If none found, report "No DRY violations found."

#### Agent 2: Security Review

> You are reviewing branch changes for security vulnerabilities.
>
> Check every changed file for:
> - Injection (SQL, command, template, path traversal)
> - XSS (unescaped user input in HTML/JSX/templates)
> - Auth/authz gaps
> - Hardcoded secrets, API keys, tokens
> - Insecure patterns, unsafe deserialization
> - Missing input validation at system boundaries
> - OWASP Top 10
>
> For each finding: file, lines, vulnerability type, severity (critical/high/medium/low), suggested fix, auto-fixable (yes only if fix is unambiguous).
> If none found, report "No security issues found."

#### Agent 3: Supportability Review

> You are reviewing branch changes for supportability — can someone debug and operate this code in production?
>
> Check every changed file for:
> - Error messages: Are they actionable? Do they include context (what failed, what was expected)?
> - Logging: Are key operations logged? Can you trace a request through the system?
> - Error handling: Are errors caught with useful context, or do they bubble as cryptic stack traces?
> - Failure modes: What happens when external calls fail? Is there graceful degradation?
>
> For each finding: file, lines, issue, severity, suggested fix, auto-fixable (yes for adding error messages or try/catch with logging).
> If none found, report "No supportability issues found."

#### Agent 4: Maintainability Review

> You are reviewing branch changes for maintainability — will the next developer understand and safely modify this code?
>
> Check every changed file for:
> - Unclear or misleading names
> - Functions >50 lines or deeply nested (>3 levels)
> - Magic numbers/strings without named constants
> - Tight coupling that blocks testing or reuse
> - Missing TypeScript types (any, implicit any)
> - Dead code, unused imports, unreachable branches
> - Leftover console.log / debugger statements
> - TODO/FIXME/HACK without issue references
>
> For each finding: file, lines, issue, severity, suggested fix, auto-fixable (yes for: unused imports, console.log removal, magic number extraction).
> If none found, report "No maintainability issues found."

#### Agent 5: Documentation Review

> You are reviewing branch changes for documentation gaps and inaccuracies.
>
> Check:
> 1. **README**: Does it still accurately describe the project? Are new features/commands/options documented?
> 2. **Doc files** (docs/, *.md): Are any now outdated or incorrect because of the code changes?
> 3. **Code comments**: Are existing comments still accurate? Are complex new sections missing explanations?
> 4. **API surface**: If public APIs changed, are type definitions / JSDoc / OpenAPI specs updated?
> 5. **Examples**: Do code examples in docs still work?
>
> For each finding: file, lines, what's wrong or missing, suggested content. All documentation fixes are auto-fixable.
> If none found, report "No documentation issues found."

#### After All Agents Complete

1. **Collect** all findings from the 5 agents
2. **Auto-fix** findings marked as auto-fixable — the fix must be unambiguous and must not change public API or behavior
3. **Re-run tests** after fixes to ensure nothing broke
4. **Commit** if any fixes were applied:
   ```
   chore: review fixes — <list categories fixed>

   Co-Authored-By: Claude <noreply@anthropic.com>
   ```
5. **Push**: `git push`
6. **Create GH issues** for remaining non-auto-fixable findings, one per category:
   ```bash
   gh issue create --title "Review: <Category> findings from <branch>" --label "review:<category>" --body "<findings>"
   ```
   Skip categories with zero remaining findings.

### Phase 8: Merge

1. Check CI: `gh pr checks <pr-number> --watch`
   - No CI or all pass: proceed
   - Fails: fix (max 2 attempts), then ask user
2. Merge: `gh pr merge <pr-number> --squash --delete-branch`

### Phase 9: Teardown

1. `cd` back to the **original working directory** (recorded in Phase 1)
2. Remove the worktree:
   ```bash
   git worktree remove <worktree-path>
   ```
3. If the local branch still exists, delete it:
   ```bash
   git branch -d <branch-name>
   ```

### Phase 10: Report

```
Done. Issue #<number> merged via PR #<pr-number>.
Branch: <branch-name>
Changes: <1-2 sentence summary>
Tests: <created N new tests / ran N existing tests — all pass>
Review: <N auto-fixes applied, N issues created>
Worktree cleaned up.
```

## Validation

- [ ] Issue fetched and understood
- [ ] Branch and worktree created
- [ ] Implementation addresses requirements
- [ ] Tests exist and all pass (created if missing)
- [ ] Committed with conventional commit message
- [ ] PR created linking to issue
- [ ] 5 parallel reviews completed
- [ ] Auto-fixes applied and pushed
- [ ] GH issues created for remaining findings
- [ ] PR merged
- [ ] Worktree removed
- [ ] Returned to original working directory

## Troubleshooting

**`gh` not found**: Install GitHub CLI: https://cli.github.com/
**CI fails after fixes**: Ask user — may be infrastructure issue
**Merge conflicts**: Ask user — needs human judgment
**Vague issue**: Ask ONE clarifying question, then proceed
**No test framework detected**: Default to vitest (Node.js) or pytest (Python); install if needed
