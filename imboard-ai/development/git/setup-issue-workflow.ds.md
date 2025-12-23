---
authors:
- name: Yuval Dimnik <yuval.dimnik@gmail.com>
checksum:
  algorithm: sha256
  hash: f97bc0be6ca3ec92bd386992c75ecd004a52acf98aedb61b145ac640177e934c
name: setup-issue-workflow
objective: Create a workflow for GitHub issues that fetches issue details, creates
  appropriately named branches, optionally sets up git worktrees, and generates planning
  files for structured development
schema_version: 1.0.0
signature:
  algorithm: ed25519
  public_key: rwZMHabZOn44qGc9tIRVPjFsHpoB3KxbsLhoULI5Xrw=
  signature: y7UlXEQNilCW/NMu7G0P0DSsylX38wHS84uNgoK/vM3Cxskk3iZNdWMIzdoZXQOJoG3eoapJuW92E1QxC7P5Dw==
  signed_by: Yuval Dimnik <yuval.dimnik@gmail.com>
  timestamp: '2025-12-23T20:29:25.105866+00:00'
status: draft
title: Setup Issue Workflow
version: 1.0.0
---

# Setup Issue Workflow

## Objective

Create a workflow for GitHub issues that fetches issue details, creates appropriately named branches, optionally sets up git worktrees, and generates planning files for structured development.

## Prerequisites

- [ ] Git is installed and configured
- [ ] GitHub CLI (gh) is installed and authenticated
- [ ] You are in a git repository with GitHub as a remote
- [ ] You have push access to create branches

## Context to Gather

### 1. Issue Information

Collect the GitHub issue number from user input. Then fetch:
- Issue title
- Issue labels (especially "bug" or "feature")
- Issue body/description
- Issue assignees (optional)

### 2. Worktree Location Discovery

Discover where to create the worktree using this priority order:

1. **Check documentation first** (avoid trial-and-error):
   - Look in `WORKTREES.md`, `.claude.md`, `agents.md`, `README.md`, `CONTRIBUTING.md` for worktree instructions
   - These files should have explicit guidance on worktree location patterns
   - **Critical**: Some repos use nested main worktree (e.g., `repo/main/`) where worktrees should be siblings to `main/`, not to `repo/`

2. **Check for sibling worktrees** (most common pattern):
   ```bash
   git worktree list
   ```

   Example output for flat structure:
   ```
   /home/user/projects/dossier     abc1234 [main]
   /home/user/projects/cli-work    def5678 [cli-work]
   /home/user/projects/feature-42  ghi9012 [feature/42-add-dashboard]
   ```

   Example output for nested main worktree:
   ```
   /home/user/projects/monorepo/main        abc1234 [main]
   /home/user/projects/monorepo/cli-work    def5678 [cli-work]
   /home/user/projects/monorepo/feature-42  ghi9012 [feature/42-add-dashboard]
   ```

   **Analysis strategy**:
   - Get the main worktree path (look for `[main]` or `[master]` branch)
   - Check if main worktree ends with `/main/` or `/master/` - indicates nested structure
   - If nested: worktrees are siblings to the nested main directory
   - If flat: worktrees are siblings to the repository root
   - For branch `bug/123-fix-login`:
     - Nested: `monorepo/bug-123-fix-login/` (sibling to `monorepo/main/`)
     - Flat: `bug-123-fix-login/` (sibling to repository root)

3. **Check for nested worktrees directory**:
   - Look for `../worktrees/` or `.worktrees/` patterns in existing worktrees
   - If found, use that directory

4. **If unclear**: Prompt user:
   ```
   ? Where should I create the worktree? (e.g., ../feature-123):
   ```

## Actions to Perform

### Step 1: Get Issue Number

Prompt the user for the GitHub issue number if not already provided:
```
? GitHub issue number:
```

### Step 2: Fetch Issue Details

Use the GitHub CLI to fetch issue information:

```bash
gh issue view <ISSUE_NUMBER> --json title,labels,body,assignees
```

Extract:
- `title` - Issue title for branch naming
- `labels` - Check for "bug" or "feature" labels
- `body` - Issue description for PLANNING.md

### Step 3: Determine Branch Type

Based on issue labels:

1. **If "bug" label present**: Use `bug/` prefix
2. **If "feature" label present**: Use `feature/` prefix
3. **If both or neither**: Prompt user to choose:
   ```
   ? Issue type:
     1) bug - This is a bug fix
     2) feature - This is a new feature
   Choice (1-2):
   ```

### Step 4: Create Branch Name

Generate the branch name:

1. **Slugify the title**:
   - Convert to lowercase
   - Replace spaces with hyphens
   - Remove special characters
   - Truncate to reasonable length (50 chars max)

2. **Construct branch name**:
   ```
   {type}/{issue-number}-{slugified-title}
   ```

   Examples:
   - `bug/123-fix-login-redirect-issue`
   - `feature/456-add-user-dashboard`

### Step 5: Choose Workflow Mode

Ask the user where they want to work on this issue:

```
? Where do you want to work on this issue?
  1) Create a git worktree (recommended for parallel work)
  2) Current directory (just create branch and planning file here)
  3) Custom location (specify your own path)
Choice (1-3):
```

If user chooses option 3, prompt for the path:
```
? Enter the path where you want to work:
```

Based on the choice:
- **Option 1 (Worktree)**: Continue with Step 6 (Discover Worktree Location)
- **Option 2 (Current directory)**: Skip to Step 7 (Create Git Branch) with checkout
- **Option 3 (Custom path)**: Skip to Step 7 (Create Git Branch), then create/navigate to custom path

### Step 6: Discover Worktree Location (Worktree Mode Only)

**Skip this step if user chose option 2 or 3.**

Follow the discovery process from "Context to Gather" section:

1. **Check documentation first**: Look in `WORKTREES.md`, `.claude.md`, etc. for explicit instructions
2. **Analyze `git worktree list`** output:
   - Check if main worktree path ends with `/main/` or `/master/` (nested structure)
   - If nested: create worktree as sibling to main (e.g., `../bug-123/` next to `../main/`)
   - If flat: create worktree as sibling to repo root
3. **Check for nested worktrees directory** pattern
4. **If unclear**: Ask user for location

### Step 7: Create Git Branch

**For Worktree mode (option 1)**:
Create the branch without checking it out (the worktree will handle checkout):

```bash
git branch <branch-name>
```

**For Current directory or Custom path mode (options 2 or 3)**:
Create and checkout the branch:

```bash
git checkout -b <branch-name>
```

Verify branch was created:
```bash
git branch --list <branch-name>
```

### Step 8: Create Git Worktree (Worktree Mode Only)

**Skip this step if user chose option 2 or 3.**

```bash
git worktree add <worktree-path> <branch-name>
```

This command checks out the branch in the new worktree directory.

Verify worktree was created:
```bash
git worktree list
```

**For Custom path mode (option 3)**:
If the custom path doesn't exist, create it:
```bash
mkdir -p <custom-path>
```

### Step 9: Generate Planning File

Create the planning file using the format `PLANNING-{issue-number}-{slug}.md`:

**Location based on workflow mode:**
- **Worktree mode**: Create in worktree root
- **Current directory mode**: Create in current directory
- **Custom path mode**: Create in the custom path specified

```markdown
# Issue #<NUMBER>: <TITLE>

## Type
<bug|feature>

## Problem Statement
<Issue body content here>

## Implementation Checklist
- [ ] Understand the issue and gather context
- [ ] Identify files that need modification
- [ ] Implement the changes
- [ ] Add/update tests
- [ ] Update documentation if needed
- [ ] Self-review the changes
- [ ] Create pull request

## Files to Modify
<!-- List the files you expect to modify -->

## Testing Strategy
<!-- Describe how you will test the changes -->
- [ ] Unit tests
- [ ] Integration tests
- [ ] Manual testing

## Notes
<!-- Additional notes, questions, or considerations -->

## Related Issues/PRs
- Issue: #<NUMBER>
```

### Step 10: Display Results

Show a summary based on the workflow mode chosen:

**For Worktree mode:**
```
Issue workflow setup complete!

Issue:      #<NUMBER> - <TITLE>
Type:       <bug|feature>
Branch:     <branch-name>
Worktree:   <worktree-path>
Planning:   <worktree-path>/PLANNING-<NUMBER>-<slug>.md

Next steps:
1. Navigate to the worktree:
   cd <worktree-path>

2. Review and update the planning file with your implementation plan

3. Start working on the issue!

4. When done, create a PR:
   gh pr create --title "<title>" --body "Closes #<NUMBER>"
```

**For Current directory mode:**
```
Issue workflow setup complete!

Issue:      #<NUMBER> - <TITLE>
Type:       <bug|feature>
Branch:     <branch-name> (checked out)
Planning:   ./PLANNING-<NUMBER>-<slug>.md

Next steps:
1. Review and update the planning file with your implementation plan

2. Start working on the issue!

3. When done, create a PR:
   gh pr create --title "<title>" --body "Closes #<NUMBER>"
```

**For Custom path mode:**
```
Issue workflow setup complete!

Issue:      #<NUMBER> - <TITLE>
Type:       <bug|feature>
Branch:     <branch-name> (checked out)
Location:   <custom-path>
Planning:   <custom-path>/PLANNING-<NUMBER>-<slug>.md

Next steps:
1. Navigate to your workspace:
   cd <custom-path>

2. Review and update the planning file with your implementation plan

3. Start working on the issue!

4. When done, create a PR:
   gh pr create --title "<title>" --body "Closes #<NUMBER>"
```

## Validation

**All modes:**
- [ ] Issue details were successfully fetched from GitHub
- [ ] Branch name follows the convention: `{type}/{number}-{slug}`
- [ ] Git branch was created successfully
- [ ] Planning file (PLANNING-{number}-{slug}.md) was created
- [ ] Planning file contains all required sections
- [ ] User was shown clear next steps

**Worktree mode only:**
- [ ] Git worktree was created at the correct location
- [ ] Planning file is in the worktree root

**Current directory mode only:**
- [ ] Branch was checked out
- [ ] Planning file is in current directory

**Custom path mode only:**
- [ ] Branch was checked out
- [ ] Custom directory was created (if needed)
- [ ] Planning file is in the custom path

## Troubleshooting

**Issue**: `gh` command not found
**Solution**: Install GitHub CLI: https://cli.github.com/

**Issue**: Not authenticated with GitHub
**Solution**: Run `gh auth login` to authenticate

**Issue**: Issue not found
**Solution**: Verify the issue number and that you have access to the repository

**Issue**: Branch already exists
**Solution**: Either use the existing branch or choose a different name

**Issue**: Worktree path already in use
**Solution**: Check `git worktree list` and choose a different location

## Examples

### Example 1: Worktree Mode

For issue #42 with title "Add user preferences page" and "feature" label:

**Input**:
```
? GitHub issue number: 42
? Where do you want to work on this issue?
  1) Create a git worktree (recommended for parallel work)
  2) Current directory (just create branch and planning file here)
  3) Custom location (specify your own path)
Choice: 1
```

**Output**:
```
Issue workflow setup complete!

Issue:      #42 - Add user preferences page
Type:       feature
Branch:     feature/42-add-user-preferences-page
Worktree:   ../feature-42-add-user-preferences-page
Planning:   ../feature-42-add-user-preferences-page/PLANNING-42-add-user-preferences-page.md

Next steps:
1. Navigate to the worktree:
   cd ../feature-42-add-user-preferences-page

2. Review and update the planning file with your implementation plan

3. Start working on the issue!

4. When done, create a PR:
   gh pr create --title "Add user preferences page" --body "Closes #42"
```

### Example 2: Current Directory Mode

For issue #99 with title "Fix login button" and "bug" label:

**Input**:
```
? GitHub issue number: 99
? Where do you want to work on this issue?
  1) Create a git worktree (recommended for parallel work)
  2) Current directory (just create branch and planning file here)
  3) Custom location (specify your own path)
Choice: 2
```

**Output**:
```
Issue workflow setup complete!

Issue:      #99 - Fix login button
Type:       bug
Branch:     bug/99-fix-login-button (checked out)
Planning:   ./PLANNING-99-fix-login-button.md

Next steps:
1. Review and update the planning file with your implementation plan

2. Start working on the issue!

3. When done, create a PR:
   gh pr create --title "Fix login button" --body "Closes #99"
```

## Notes

- This workflow assumes a GitHub-based repository
- Three workflow modes are available:
  - **Worktree mode**: Best for working on multiple issues simultaneously
  - **Current directory mode**: Simplest option, works in place
  - **Custom path mode**: For users with their own directory structure
- The planning file (PLANNING-{number}-{slug}.md) helps maintain focus and track progress
- Consider adding this workflow to your `.claude.md` for easy access

## References

- [Git Worktrees Documentation](https://git-scm.com/docs/git-worktree)
- [GitHub CLI Documentation](https://cli.github.com/manual/)
- [Branch Naming Conventions](https://www.conventionalcommits.org/)