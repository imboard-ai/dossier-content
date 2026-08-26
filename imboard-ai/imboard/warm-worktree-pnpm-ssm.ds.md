---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "warm-worktree-pnpm-ssm",
  "title": "Imboard Warm Worktree (pnpm + SSM)",
  "version": "1.3.0",
  "protocol_version": "1.0",
  "status": "Stable",
  "objective": "Prepare a fresh imboard-monorepo worktree for development using pnpm content-addressable store and AWS SSM secrets — no .env copying needed",
  "category": [
    "development"
  ],
  "tags": [
    "worktree",
    "pnpm",
    "ssm",
    "imboard"
  ],
  "estimated_duration": {
    "min_minutes": 0,
    "max_minutes": 2
  },
  "risk_level": "low",
  "requires_approval": false,
  "inputs": {
    "required": [
      {
        "name": "target_worktree",
        "description": "Path to the newly created worktree to warm up",
        "type": "string"
      }
    ]
  },
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "23ef71c54e774c5d74628773a203fb84caba8a0280b5a06e09ad12dcc901aa86"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "15qhlerkV9nXy+UMxNDBqjlcF45Oc6f+5q4Ven+/LAIykq+6X4k2hkdpszzEoNHUtVA7vL1t2A6XIguIMq43CA==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-28T08:19:39.320Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Imboard Warm Worktree (pnpm + SSM)

Prepare a fresh imboard-monorepo worktree for development. With pnpm's content-addressable store and secrets on AWS SSM, warmup takes ~15 seconds total.

## Inputs

- **target_worktree**: Path to the newly created worktree

## Actions

### Step 1: Install dependencies

```bash
cd <target_worktree>
pnpm install
```

pnpm hard-links packages from its global content-addressable store, so this completes in ~3-5 seconds for repeat installs.

### Step 2: Build workspace libraries

```bash
pnpm run build:libs
```

This runs `build:shared` + `build:emails` — the workspace library packages (`@imboard/shared-types`, `@imboard/emails`) that other packages import and that must exist as `dist/` before typecheck passes. The backend imports both; the frontend imports `shared-types`. `build:libs` is defined in the root `package.json`, so the warmup never enumerates packages itself — when a new imported library package is added, only `build:libs` changes.

### Step 3: Verify (quick check)

```bash
pnpm run typecheck
```

If this passes, the worktree is ready for development.

### Step 4: Provision the pooled test environment

```bash
cd <target_worktree>/main
bash scripts/ensure-test-env.sh
```

This writes `<target_worktree>/main/.env.test` — including `IMBOARD_TEST_POOL_URI` (fetched from SSM `mongodb-pool-uri`), which is what makes `pnpm test:integration` use the shared leased test-DB pool instead of minting a fresh per-worktree database (imboard#3757/#3763). **Run this unconditionally, even if `.env.test` already exists** — a worktree reused from before this step existed, or one whose `.env.test` predates the pool migration, has a file that satisfies every downstream guard but carries no `IMBOARD_TEST_POOL_URI`. Nothing else in the chain detects that: `test-runner.ts`'s `.env.test` guard only checks the file exists, and `jest.globalSetup.ts` silently falls back to the legacy isolated-DB path with no warning when the pool var is absent. Re-running `ensure-test-env.sh` is idempotent and safe to repeat.

If this step fails (e.g. `ssm_get` can't reach the pool secret), warmup is still usable for non-test work — but say so explicitly rather than silently continuing, and do not assume tests will use the pool.

## What This Does NOT Do (and why)

- **No general secret copying** — App secrets (SSM-backed) come via `chamber exec`. Run `pnpm run dev:ssm` to start servers with injected secrets. (`.env.test` is the one exception — Step 4 provisions it directly, since the pooled test path needs it present before any test command runs.)
- **No server startup verification** — `dev:ssm` handles secret injection at runtime; verifying here adds no value.
- **No full test suite** — Testing happens after implementation, not during warmup.
- **No worktree pool for the warmed worktree itself** — With pnpm, cold starts are ~15 seconds. Worktree-pool infrastructure adds complexity for marginal gain here. (Unrelated to the *test-DB* pool that Step 4 wires up — same word, different pool.)

## Validation

- [ ] `pnpm install` completed without errors
- [ ] `pnpm run build:libs` completed without errors
- [ ] `pnpm run typecheck` passes (all packages)
- [ ] `main/.env.test` exists and contains `IMBOARD_TEST_POOL_URI` (or Step 4's failure was surfaced explicitly, not silently skipped)

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `pnpm: command not found` | Install: `npm install -g pnpm` or use `corepack enable` |
| Node version mismatch | This project uses `packageManager: pnpm@10.32.1` — ensure compatible Node |
| `build:libs` fails | Check for uncommitted type changes in `shared-types`/`emails` on the source branch |
| typecheck fails | May indicate pre-existing errors on main — check `git log` for recent changes |
| `ensure-test-env.sh` fails on `ssm_get mongodb-pool-uri` | Check SSM/chamber access is configured for this environment; report it rather than proceeding with an unpooled `.env.test` unremarked |
