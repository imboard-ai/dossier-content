---
schema_version: "1.0.0"
name: warm-worktree
title: Warm Worktree
version: "1.0.0"
status: stable
objective: Prepare a fresh git worktree for development by copying environment files, installing dependencies, running builds, verifying tests, and checking that servers can start.
authors:
  - name: Dossier Community
checksum:
  algorithm: sha256
  hash: 5a11a58547595a1435cc62a2b5ba18594e3fe9834633b286da423c009c36541e
risk_level: low
risk_factors:
  - executes_external_code
  - network_access
requires_approval: false
category:
  - development
  - git
tags:
  - worktree
  - setup
  - environment
  - dependencies
estimated_duration:
  min_minutes: 1
  max_minutes: 10
---

# Warm Worktree

Prepare a fresh git worktree for development by setting up the environment, installing dependencies, and verifying the project builds and runs correctly.

## Inputs

- **source_worktree**: Path to source worktree (where .env files and working environment exist)
- **target_worktree**: Path to newly created worktree to warm up

## Output

- `WARMUP-STATUS.md` file in target worktree tracking progress and any issues

---

## Step 1: Create Status File

Create `WARMUP-STATUS.md` in the target worktree root to track progress.

**Location:** `<target_worktree>/WARMUP-STATUS.md`

**Ensure not committed:**
```bash
# Add to local git exclude (doesn't modify .gitignore)
echo "WARMUP-STATUS.md" >> <target_worktree>/.git/info/exclude
```

**Initial content:**
```markdown
# Worktree Warmup Status

**Source:** <source_worktree>
**Target:** <target_worktree>
**Started:** <current timestamp>
**Status:** IN_PROGRESS

## Steps

| Step | Status | Duration | Notes |
|------|--------|----------|-------|
| Copy .env files | ⏳ Pending | - | - |
| Install dependencies | ⏳ Pending | - | - |
| Build | ⏳ Pending | - | - |
| Tests | ⏳ Pending | - | - |
| Servers | ⏳ Pending | - | - |
```

Update this file after each step completes.

---

## Step 2: Copy .env Files

Copy environment files from source worktree to target worktree.

### Detection

Find all `.env*` files in source worktree (recursively):
```bash
find <source_worktree> -name ".env*" -type f
```

**Patterns to match:**
- `.env`
- `.env.local`
- `.env.development`
- `.env.development.local`
- `.env.test`
- `.env.production.local`
- Nested paths: `apps/api/.env`, `packages/db/.env`

**Exclude:**
- `.env.example` (template files, already in git)
- Files inside `node_modules/`, `.git/`, `vendor/`

### Action

For each .env file found:
1. Determine relative path from source root
2. Create parent directories in target if needed
3. Copy file to same relative path in target

```bash
# Example
cp <source>/apps/api/.env <target>/apps/api/.env
```

### On Success

Update status file:
```
| Copy .env files | ✅ Done | 0.2s | Copied N files |
```

List copied files in Notes section.

### On Failure

Update status file with error:
```
| Copy .env files | ❌ Failed | 0.1s | See errors below |
```

Add to Errors section:
```markdown
## Errors

### Copy .env Files Failed

Could not copy the following files:
- `apps/api/.env`: Permission denied
- `packages/db/.env`: Directory does not exist

**Suggested Fix:**
- Check file permissions in source worktree
- Ensure parent directories exist in target worktree
```

---

## Step 3: Install Dependencies

Detect package manager(s) and install dependencies.

### Detection

Check for these files in target worktree (in order):

| File | Package Manager | Command |
|------|-----------------|---------|
| `pnpm-lock.yaml` | pnpm | `pnpm install` |
| `yarn.lock` | yarn | `yarn install` |
| `package-lock.json` | npm | `npm install` |
| `bun.lockb` | bun | `bun install` |
| `package.json` (no lock) | npm | `npm install` |
| `uv.lock` | uv | `uv sync` |
| `pyproject.toml` | pip/uv | `uv sync` or `pip install -e .` |
| `requirements.txt` | pip | `pip install -r requirements.txt` |
| `go.mod` | go | `go mod download` |
| `Cargo.toml` | cargo | `cargo fetch` |
| `Gemfile` | bundler | `bundle install` |
| `composer.json` | composer | `composer install` |

**For monorepos:** A project may have multiple package managers (e.g., Node at root, Python in `services/ml/`). Run all detected installers.

### Action

Run the install command in the target worktree:
```bash
cd <target_worktree>
<install_command>
```

### On Success

Update status file:
```
| Install dependencies | ✅ Done | 45s | npm install |
```

### On Failure

Capture full error output. Common issues and suggested fixes:

| Error Pattern | Likely Cause | Suggested Fix |
|--------------|--------------|---------------|
| `EACCES` | Permission issue | Check npm cache permissions |
| `401 Unauthorized` | Private registry auth | Run `npm login` or check `.npmrc` |
| `node: command not found` | Node not installed | Install Node.js or use `nvm` |
| `ERESOLVE` | Dependency conflict | Try `npm install --legacy-peer-deps` |
| `ModuleNotFoundError` | Missing Python deps | Check virtual environment is active |

Update status file:
```
| Install dependencies | ❌ Failed | 12s | See errors below |
```

Add error details and suggested fix to Errors section.

---

## Step 4: Run Build

Run the project build command if one exists.

### Detection

Check for build commands:

| Location | Key | Command |
|----------|-----|---------|
| `package.json` scripts | `build` | `npm run build` |
| `package.json` scripts | `build:dev` | `npm run build:dev` |
| `Makefile` | `build` target | `make build` |
| `pyproject.toml` | build-system | `python -m build` |
| `Cargo.toml` | (Rust project) | `cargo build` |
| `go.mod` | (Go project) | `go build ./...` |

If no build command detected, skip this step and note in status file.

### Action

Run the build command:
```bash
cd <target_worktree>
<build_command>
```

### On Success

Update status file:
```
| Build | ✅ Done | 30s | npm run build |
```

### On Failure

Capture error output. Analyze for common issues:

| Error Type | Pattern | Suggested Fix |
|------------|---------|---------------|
| TypeScript | `error TS\d+` | Show file:line, suggest type fix |
| ESLint | `error.*eslint` | Run `npm run lint -- --fix` |
| Missing env var | `process.env.X is undefined` | Check .env file was copied |
| Import error | `Cannot find module` | Dependencies may not be installed |

Update status file with error and suggested fix.

---

## Step 5: Run Tests (User Choice)

Ask user whether to run tests for baseline verification.

### Prompt User

```
? Run tests to establish baseline?
  1) Skip (fastest)
  2) Smoke tests only (if available)
  3) Full test suite
Choice:
```

### Detection

| Location | Key | Command |
|----------|-----|---------|
| `package.json` scripts | `test` | `npm test` |
| `package.json` scripts | `test:smoke` | `npm run test:smoke` |
| `package.json` scripts | `test:unit` | `npm run test:unit` |
| `Makefile` | `test` target | `make test` |
| `pyproject.toml` | pytest | `pytest` |
| `Cargo.toml` | (Rust) | `cargo test` |
| Directory | `tests/`, `__tests__/` | Auto-detect runner |

### Action

Based on user choice:
- **Skip**: Update status and continue
- **Smoke**: Run smoke test command if available, else skip
- **Full**: Run full test suite

### On Success

Update status file:
```
| Tests | ✅ Done | 2m 15s | 42 passed, 0 failed |
```

### On Failure

Note that these failures exist on the base branch, not from user's changes:

```markdown
## Errors

### Tests Failed (Baseline)

**Important:** These test failures exist on the base branch (main/master).
They are NOT caused by your changes.

Failed tests:
- `src/api/users.test.ts`: Expected 200, got 401
- `src/utils/date.test.ts`: Timezone mismatch

**Suggested Actions:**
1. Check if these tests are known failures (check CI status on main)
2. Create an issue for broken tests if not already tracked
3. Continue with your work - these failures are pre-existing
```

Update status file:
```
| Tests | ⚠️ Failed | 1m 30s | Baseline failures (see errors) |
```

---

## Step 6: Verify Servers

Start each detected server, verify it responds, then stop it.

### Detection

| Location | Key | Purpose |
|----------|-----|---------|
| `package.json` scripts | `dev` | Development server |
| `package.json` scripts | `start` | Production server |
| `package.json` scripts | `serve` | Static server |
| `docker-compose.yml` | services | Docker services |
| `Makefile` | `run`, `serve`, `dev` | Server targets |

### Action

For each detected server:

1. **Start in background:**
   ```bash
   cd <target_worktree>
   <start_command> &
   SERVER_PID=$!
   ```

2. **Wait for ready signal** (up to 30 seconds):
   - Check if expected port becomes available
   - Common ports: 3000, 3001, 5000, 8000, 8080
   - Or check health endpoint: `/health`, `/api/health`, `/healthz`

3. **Stop server:**
   ```bash
   kill $SERVER_PID
   ```

4. **Report result**

### On Success

Update status file:
```
| Servers | ✅ Done | 5s | dev server started on :3000 |
```

### On Failure

Capture startup error. Common issues:

| Error Pattern | Likely Cause | Suggested Fix |
|--------------|--------------|---------------|
| `EADDRINUSE` | Port in use | Kill process on port or use different port |
| `ECONNREFUSED` | Database not running | Start database (docker-compose up db) |
| `Missing env var` | .env not copied | Check Step 2 completed |
| `Cannot find module` | Build not run | Check Step 4 completed |

Update status file:
```
| Servers | ❌ Failed | 30s | See errors below |
```

---

## Step 7: Final Report

Update status file with final summary.

### All Passed

```markdown
## Summary

✅ **Worktree is ready for development!**

All checks passed:
- .env files copied (3 files)
- Dependencies installed (npm)
- Build successful
- Tests passed (or skipped by user)
- Servers verified (dev on :3000)

**You can start coding.**
```

Update status: `**Status:** COMPLETED`

### Some Failed

```markdown
## Summary

⚠️ **Worktree warmup completed with issues.**

Passed:
- .env files copied
- Dependencies installed

Failed:
- Build: TypeScript errors
- Tests: Skipped (blocked by build)
- Servers: Skipped (blocked by build)

## Suggested Actions

1. Fix TypeScript error in `src/api/users.ts:42`
   - Property 'foo' does not exist on type 'User'
   - Add the property to the User interface or remove the reference

2. Re-run warmup after fixing:
   ```bash
   dossier run imboard-ai/development/git/warm-worktree
   ```

## Options

1. Fix the issues above and re-run warmup
2. Continue anyway (issues are noted above)
3. Abort and investigate the base branch
```

Update status: `**Status:** FAILED`

---

## Notes

- The status file `WARMUP-STATUS.md` is excluded from git commits
- This dossier is typically called from `setup-issue-workflow` after creating a worktree
- Can also be run standalone to re-warm an existing worktree
- Failures in later steps (tests, servers) don't block earlier steps from completing
- Build failures may indicate problems on the base branch, not the user's work
