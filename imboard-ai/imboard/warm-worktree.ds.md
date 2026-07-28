---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "warm-worktree",
  "title": "Imboard Warm Worktree (pnpm + SSM)",
  "version": "1.1.3",
  "protocol_version": "1.0",
  "status": "Deprecated",
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
    "max_minutes": 1
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
    "hash": "f01c718a890c357e5654868589336a0ad01d6b9f6fc22b39867b276ec4b99285"
  },
  "relationships": {
    "alternatives": [
      {
        "dossier": "imboard-ai/imboard/warm-worktree-pnpm-ssm",
        "when_to_use": "Always — this is the maintained location."
      }
    ]
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "I1LA2ijKTHoriHDiilnPLEsw3k8ASrfudUpioDTwoRPEJPB0FMywk38eFpoGGjHBeS484whETB3o6PoCJansAg==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-28T08:19:28.933Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---
> **Deprecated — moved to `imboard-ai/imboard/warm-worktree-pnpm-ssm`.**
>
> Both paths installed to the same skill directory (`install-skill` keys on the
> basename), so whichever was installed last silently replaced the other. This copy
> is no longer maintained. Use `imboard-ai/imboard/warm-worktree-pnpm-ssm`.


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

## What This Does NOT Do (and why)

- **No .env file copying** — Secrets come from AWS SSM via `chamber exec`. Run `pnpm run dev:ssm` to start servers with injected secrets.
- **No server startup verification** — `dev:ssm` handles secret injection at runtime; verifying here adds no value.
- **No full test suite** — Testing happens after implementation, not during warmup.
- **No worktree pool** — With pnpm, cold starts are ~15 seconds. Pool infrastructure adds complexity for marginal gain.

## Validation

- [ ] `pnpm install` completed without errors
- [ ] `pnpm run build:libs` completed without errors
- [ ] `pnpm run typecheck` passes (all packages)

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `pnpm: command not found` | Install: `npm install -g pnpm` or use `corepack enable` |
| Node version mismatch | This project uses `packageManager: pnpm@10.32.1` — ensure compatible Node |
| `build:libs` fails | Check for uncommitted type changes in `shared-types`/`emails` on the source branch |
| typecheck fails | May indicate pre-existing errors on main — check `git log` for recent changes |
