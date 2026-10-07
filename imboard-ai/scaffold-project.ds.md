---
name: 'scaffold-project'
description: 'Scaffold a complete TypeScript project with CI, testing, linting, documentation, and worktree support — eliminating repetitive boilerplate setup'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Scaffold TypeScript Project'
  dossier.version: '1.0.2'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.last_updated: '2026-03-09'
  dossier.objective: 'Scaffold a complete TypeScript project with CI, testing, linting, documentation, and worktree support — eliminating repetitive boilerplate setup'
  dossier.category: '["development","setup"]'
  dossier.tags: '["scaffold","typescript","boilerplate","ci","project-creation","github-actions"]'
  dossier.tools_required: '[{"check_command":"node --version","name":"node","version":">=20.0.0"},{"check_command":"gh --version","name":"gh","version":">=2.0.0"},{"check_command":"git --version","name":"git","version":">=2.30.0"}]'
  dossier.estimated_duration: '{"max_minutes":15,"min_minutes":5}'
  dossier.risk_level: 'medium'
  dossier.risk_factors: '["modifies_files"]'
  dossier.requires_approval: 'false'
  dossier.destructive_operations: '["Creates multiple files in the target directory","Runs npm install (modifies node_modules and package-lock.json)"]'
  dossier.content_scope: 'references-external'
  dossier.external_references: '[{"description":"Documentation link mentioned in the body: biomejs.dev","required":false,"trust_level":"trusted","type":"documentation","url":"https://biomejs.dev/schemas/2.0.0/schema.json"}]'
  dossier.inputs: '{"optional":[{"default":"","description":"GitHub organization for the repo (skip if repo already exists)","example":"imboard-ai","name":"github_org","type":"string"},{"default":"MIT","description":"License type","example":"AGPL-3.0","name":"license","type":"string"},{"default":"biome","description":"Linter to use: biome or eslint","example":"eslint","name":"linter","type":"string"},{"default":"22","description":"Node.js version for CI matrix","example":"20","name":"node_version","type":"string"},{"default":false,"description":"Skip worktree support setup","name":"skip_worktrees","type":"boolean"},{"default":false,"description":"Skip GitHub repo creation (use if repo already exists)","name":"skip_github_repo","type":"boolean"},{"default":"","description":"Author name for package.json","example":"Yuval Dimnik","name":"author_name","type":"string"},{"default":"","description":"Author email for package.json","example":"yuval.dimnik@gmail.com","name":"author_email","type":"string"}],"required":[{"description":"Name of the project (kebab-case, used for package.json name and repo)","example":"my-awesome-tool","name":"project_name","type":"string"},{"description":"Absolute path to the project root directory","example":"/home/user/projects/my-awesome-tool/main","name":"project_dir","type":"string"},{"description":"One-line project description","example":"CLI tool for managing AI agent workflows","name":"description","type":"string"}]}'
  dossier.outputs: '{"files":[{"description":"Package manifest with ESM, scripts, devDependencies","path":"package.json"},{"description":"TypeScript config (strict, ES2022, ESNext modules)","path":"tsconfig.json"},{"description":"Comprehensive gitignore for Node/TypeScript","path":".gitignore"},{"description":"GitHub Actions CI (typecheck + lint + test)","path":".github/workflows/ci.yml"},{"description":"AI agent behavioral rules and project context","path":"AGENTS.md"},{"description":"Vitest test runner configuration","path":"vitest.config.ts"},{"description":"Example environment variables","path":".env.example"},{"description":"Entry point placeholder","path":"lib/index.ts"}]}'
  dossier.checksum: '{"algorithm":"sha256","hash":"7efc61ddb1b9b1310a07c2e879e96a02ba651fd1eb35bd2317fe8869875a40c3"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"c8ARaEBq95DkevpIQw5TSZg2SF+3XxbTw+6WFktuyCOfhMuoziVBOrfELXj7dBOwEa1vfjRP5/79+ZuViqd4Ag==","signed_at":"2026-10-07T12:11:24.586Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---
# Scaffold TypeScript Project

## Objective

Scaffold a complete TypeScript project with all standard boilerplate: CI pipeline, testing, linting, documentation, environment management, and optionally worktree support. This eliminates the repetitive "let me add CI, let me add workflows, let me add configs" cycle when starting new projects.

## Prerequisites

- [ ] Node.js >= 20 installed
- [ ] GitHub CLI authenticated (`gh auth status`)
- [ ] Git configured with user name and email
- [ ] Target directory exists or can be created

## Context to Gather

Before scaffolding:

1. **Check if directory exists** — if it has existing files, warn and ask before overwriting
2. **Check if git repo exists** — if `.git/` already exists, skip `git init`
3. **Check if GitHub repo exists** — if `skip_github_repo` is false, check with `gh repo view ${github_org}/${project_name}` before creating
4. **Detect author info** — if `author_name`/`author_email` not provided, read from `git config user.name` and `git config user.email`

## Actions to Perform

### Step 1: Create Directory Structure

```
${project_dir}/
├── lib/                    # Source code
│   ├── index.ts            # Entry point
│   └── __tests__/          # Test files
├── .github/
│   └── workflows/
│       └── ci.yml          # CI pipeline
├── package.json
├── tsconfig.json
├── vitest.config.ts
├── .gitignore
├── .env.example
├── AGENTS.md
└── README.md
```

Create the directory structure:
```bash
mkdir -p ${project_dir}/{lib/__tests__,.github/workflows}
```

### Step 2: Create package.json

```json
{
  "name": "${project_name}",
  "version": "0.1.0",
  "description": "${description}",
  "type": "module",
  "scripts": {
    "build": "tsc",
    "dev": "node --env-file=.env --import tsx lib/index.ts",
    "test": "vitest run",
    "test:watch": "vitest",
    "typecheck": "tsc --noEmit",
    "lint": "${linter === 'biome' ? 'biome check .' : 'eslint src/'}",
    "lint:fix": "${linter === 'biome' ? 'biome check --write .' : 'eslint src/ --fix'}",
    "check": "npm run typecheck && npm run lint && npm run test"
  },
  "author": "${author_name} <${author_email}>",
  "license": "${license}",
  "engines": {
    "node": ">=${node_version}.0.0"
  },
  "devDependencies": {}
}
```

Then install dev dependencies:
```bash
cd ${project_dir}
npm install --save-dev typescript @types/node tsx vitest
```

If `linter === 'biome'`:
```bash
npm install --save-dev @biomejs/biome
```

If `linter === 'eslint'`:
```bash
npm install --save-dev eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

### Step 3: Create tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "outDir": "dist",
    "rootDir": ".",
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["lib/**/*.ts"],
  "exclude": ["node_modules", "dist", "lib/__tests__"]
}
```

### Step 4: Create .gitignore

```
# Dependencies
node_modules/

# Build output
dist/
build/
*.tsbuildinfo

# Environment
.env
.env.*
!.env.example

# Logs
logs/
*.log
npm-debug.log*

# Coverage & test output
coverage/
test-results/

# OS files
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp

# Temp
.cache/
temp/
tmp/
```

### Step 5: Create GitHub Actions CI

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '${node_version}'
          cache: 'npm'
      - run: npm ci
      - run: npx tsc --noEmit
      - run: npm run lint
      - run: npm test
```

### Step 6: Create vitest.config.ts

```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    include: ['lib/__tests__/**/*.test.ts'],
  },
});
```

### Step 7: Create Linter Config

**If biome**: Create `biome.json`:
```json
{
  "$schema": "https://biomejs.dev/schemas/2.0.0/schema.json",
  "vcs": {
    "enabled": true,
    "clientKind": "git",
    "useIgnoreFile": true
  },
  "formatter": {
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true
    }
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "semicolons": "always",
      "trailingCommas": "es5"
    }
  }
}
```

**If eslint**: Create `eslint.config.js`:
```javascript
import tseslint from '@typescript-eslint/eslint-plugin';
import tsparser from '@typescript-eslint/parser';

export default [
  {
    files: ['lib/**/*.ts'],
    languageOptions: {
      parser: tsparser,
      parserOptions: {
        ecmaVersion: 2022,
        sourceType: 'module',
      },
    },
    plugins: {
      '@typescript-eslint': tseslint,
    },
    rules: {
      ...tseslint.configs.recommended.rules,
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    },
  },
];
```

### Step 8: Create .env.example

```bash
# ${project_name} environment variables
# Copy this file to .env and fill in values:
#   cp .env.example .env

# Example:
# API_KEY=your-api-key-here
```

### Step 9: Create AGENTS.md

```markdown
# ${project_name}

## What This Is

${description}

## Repo Structure

```
${project_name}/
├── lib/                    # Source code
│   ├── index.ts            # Entry point
│   └── __tests__/          # Tests (vitest)
├── .github/workflows/      # CI/CD
├── package.json            # ESM, TypeScript
├── tsconfig.json           # Strict TypeScript
└── AGENTS.md               # This file
```

## Build & Test

```bash
npm run typecheck           # TypeScript type checking
npm run lint                # Lint (${linter})
npm run test                # Run tests (vitest)
npm run check               # All checks (typecheck + lint + test)
npm run dev                 # Run with .env loaded
```

## Code Style

- **TypeScript**: ESM (`"type": "module"`), strict mode
- **Linter**: ${linter}
- **Tests**: vitest, files in `lib/__tests__/*.test.ts`
- **Dates**: ISO 8601 (`YYYY-MM-DD`)
- **IDs**: kebab-case
- **JSON**: 2-space indent, trailing newline

## Conventions

- Run `npm run check` before committing
- Zero lint/type errors policy — don't introduce regressions
- Unused vars: prefix with `_`
- Tests: co-located in `lib/__tests__/`, named `*.test.ts`
```

### Step 10: Create Entry Point

Create `lib/index.ts`:
```typescript
/**
 * ${project_name} — ${description}
 */

export function main(): void {
  console.log('${project_name} is running');
}

// Run if called directly
if (import.meta.url === `file://${process.argv[1]}`) {
  main();
}
```

Create `lib/__tests__/index.test.ts`:
```typescript
import { describe, it, expect } from 'vitest';
import { main } from '../index.js';

describe('${project_name}', () => {
  it('should export main function', () => {
    expect(typeof main).toBe('function');
  });
});
```

### Step 11: Create README.md

```markdown
# ${project_name}

${description}

## Quick Start

```bash
# Install dependencies
npm install

# Run
npm run dev

# Test
npm test
```

## Development

See [AGENTS.md](./AGENTS.md) for development conventions and project structure.

## License

${license}
```

### Step 12: Set Up Worktree Support (if not skipped)

If `skip_worktrees` is false:

1. Ensure the project directory is named `main/` (the main worktree)
2. Create `WORKTREES.md`:

```markdown
# Worktrees

Active worktrees for ${project_name}.

| Worktree | Branch | Created | Status | Purpose |
|----------|--------|---------|--------|---------|
| main/    | main   | ${today} | Active | Primary development |

## Commands

```bash
# Create a feature worktree
git worktree add ../feature-name -b feature/name

# List all worktrees
git worktree list

# Remove a worktree after merging
git worktree remove ../feature-name
```

## Rules

- Never checkout branches in `main/` — always create a worktree
- Each worktree gets its own `node_modules` — run `npm install` after creating
- Clean up worktrees after PR merge
```

### Step 13: Initialize Git & GitHub

If no `.git/` directory exists:
```bash
cd ${project_dir}
git init
git add -A
git commit -m "feat: scaffold ${project_name} project"
```

If `skip_github_repo` is false and `github_org` is provided:
```bash
gh repo create ${github_org}/${project_name} --private --source=. --push
```

If repo already exists, just push:
```bash
git remote add origin https://github.com/${github_org}/${project_name}.git
git push -u origin main
```

### Step 14: Verify Everything Works

Run the full check suite:
```bash
cd ${project_dir}
npm run check
```

Expected output:
- `tsc --noEmit` passes with zero errors
- Linter passes with zero errors/warnings
- `vitest run` shows 1 passing test

## Decision Points

### 1. Biome vs ESLint
- **Biome** (default): Faster, simpler config, all-in-one (lint + format). Best for new projects.
- **ESLint**: More ecosystem plugins, more granular control. Best when you need React/specific plugins.

### 2. Worktree Support
- **Enable** (default): Required for parallel agent development; recommended for any repo using multi-agent workflows.
- **Skip**: For small scripts or throwaway projects.

### 3. GitHub Repo
- **Create**: For real projects that will be maintained.
- **Skip**: For local experiments or when repo already exists.

## Validation

- [ ] `package.json` has `"type": "module"` and correct scripts
- [ ] `tsconfig.json` is strict with ESNext modules
- [ ] `.gitignore` covers node_modules, dist, .env, IDE files
- [ ] GitHub Actions CI runs typecheck + lint + test
- [ ] vitest.config.ts points to correct test directory
- [ ] Linter config exists and passes on generated code
- [ ] `.env.example` exists (even if empty)
- [ ] `AGENTS.md` describes project structure and conventions
- [ ] `README.md` has quick start instructions
- [ ] Entry point runs: `npm run dev`
- [ ] Tests pass: `npm test`
- [ ] Full check passes: `npm run check`
- [ ] Git repo initialized with initial commit
- [ ] If worktrees enabled: `WORKTREES.md` exists

## Troubleshooting

**Issue**: `npm run dev` fails with "Cannot find module"
**Solution**: Ensure `"type": "module"` in package.json and `--import tsx` flag is present in the dev script.

**Issue**: TypeScript errors on `import.meta.url`
**Solution**: Ensure `"module": "ESNext"` in tsconfig.json (not CommonJS).

**Issue**: Vitest can't find tests
**Solution**: Check `vitest.config.ts` include pattern matches your test file locations.

**Issue**: CI fails but local passes
**Solution**: Check Node version in CI matches local. Ensure `package-lock.json` is committed.
