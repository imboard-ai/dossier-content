---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "warm-worktree",
  "title": "Imboard Warm Worktree (pnpm + SSM)",
  "version": "1.0.0",
  "status": "Stable",
  "objective": "Prepare a fresh imboard-monorepo worktree for development using pnpm content-addressable store and AWS SSM secrets — no .env copying needed",
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "risk_level": "low",
  "requires_approval": false,
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
    "max_minutes": 1
  },
  "inputs": {
    "required": [
      {
        "name": "target_worktree",
        "description": "Path to the newly created worktree to warm up",
        "type": "string"
      }
    ]
  },
  "checksum": {
    "algorithm": "sha256",
    "hash": "0a789729516fd027ae6d43bb4b2dd3da48d63ba98b3647adb8c484d5f6ebb04c"
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

### Step 2: Build shared types

```bash
pnpm run build:shared
```

This runs `tsc --project packages/shared-types/tsconfig.json` to produce `dist/index.js` + `dist/index.d.ts`. Both backend and frontend depend on this output.

### Step 3: Verify (quick check)

```bash
pnpm run typecheck
```

If this passes, the worktree is ready for development.

## What This Does NOT Do (and why)

- **No .env file copying** — Secrets come from AWS SSM via `chamber exec`. Run `pnpm run dev:ssm` to start servers with injected secrets.
- **No server startup verification** — `dev:ssm` handles secret injection at runtime; verifying here adds no value.
- **No full test suite** — Testing happens after implementation, not during warmup.
- **No worktree pool** — With pnpm, cold starts are ~15 seconds. Pool infrastructure adds complexity for marginal gain.

## Validation

- [ ] `pnpm install` completed without errors
- [ ] `pnpm run build:shared` completed without errors
- [ ] `pnpm run typecheck` passes (all packages)

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `pnpm: command not found` | Install: `npm install -g pnpm` or use `corepack enable` |
| Node version mismatch | This project uses `packageManager: pnpm@10.32.1` — ensure compatible Node |
| shared-types build fails | Check for uncommitted type changes on the source branch |
| typecheck fails | May indicate pre-existing errors on main — check `git log` for recent changes |
