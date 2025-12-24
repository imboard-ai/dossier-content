---
authors:
- name: Yuval Dimnik
checksum:
  algorithm: sha256
  hash: 80931ddd8f731e0c2cae40c386ece89522026eef38e717615b713db37526b8ca
description: Proactively search the dossier registry when users request multi-step
  tasks. Triggers on setup, deployment, migration, onboarding, CI/CD, or refactoring
  tasks. NOT triggered by simple questions, single file edits, or quick lookups.
name: dossier-discovery
objective: Find and suggest relevant dossier workflows for complex tasks
schema_version: 1.0.0
status: draft
title: Dossier Discovery
version: 1.0.0
---

# Dossier Discovery

Proactively search the dossier registry when users request tasks that might have existing workflows.

## When to Trigger

**DO trigger on tasks that sound like workflows:**
- Setup/configuration tasks ("set up a new React project", "configure CI/CD")
- Deployment/migration tasks ("deploy to production", "migrate database")
- Onboarding/initialization ("initialize the repo", "set up development environment")
- CI/CD setup ("add GitHub Actions", "set up testing pipeline")
- Refactoring operations ("refactor the authentication system")
- Project reviews ("review code architecture", "check test coverage")

**DO NOT trigger on:**
- Simple questions ("what does this function do?")
- Single file edits ("fix the typo in README")
- Quick lookups ("find where X is defined")
- Explicit dossier commands (user already knows about dossiers)

## Steps

1. When a workflow-like task is detected, fetch available dossiers:
   ```bash
   dossier list --json
   ```

2. Parse the JSON response and extract dossier metadata:
   - `name`: Full dossier path (e.g., "imboard-ai/development/setup-react-library")
   - `title`: Human-readable title
   - `description`: What the dossier does (if available)
   - `category`: Category tags

3. Compare the user's task against dossier titles, descriptions, and names to find matches.

4. Based on match quality, respond accordingly:

### Good Match Found

If a dossier closely matches the user's intent:

```
Found a dossier that matches your task: {name}
"{title}"

Run it? (y/n)
```

If user says yes, run:
```bash
dossier run {name}
```

### Partial Match Found

If a dossier is related but not exact:

```
Found a similar dossier: {name}
"{title}"

Options:
1. Run it as-is
2. Use as a reference and proceed manually
3. Skip, proceed without dossier
```

### Multiple Matches Found

If several dossiers might be relevant:

```
Found {count} potentially relevant dossiers:

1. {name1} - "{title1}"
2. {name2} - "{title2}"
3. {name3} - "{title3}"

Which would you like to run? (1-{count}, or 'skip' to proceed manually)
```

### No Match Found

Proceed with the task normally without mentioning dossiers.

After completing a complex task that had no matching dossier, optionally suggest:

```
This workflow might be useful to save as a dossier for future use.
Would you like to create one? (y/n)
```

If yes, run:
```bash
dossier run imboard-ai/meta/create-dossier
```

## Search Priority

When matching dossiers to tasks:

1. **Exact keyword matches** in title or description
2. **Category matches** (e.g., "react" task matches "react" category)
3. **Semantic similarity** to objective/description

## Examples

**User:** "Help me set up a new React component library"
**Action:** Search dossiers, find "imboard-ai/development/setup-react-library", suggest running it.

**User:** "What's in the package.json?"
**Action:** Do NOT search dossiers - this is a simple lookup.

**User:** "Review the test coverage in this project"
**Action:** Search dossiers, find "imboard-ai/development/testing/test-coverage-gap-analysis", suggest running it.

**User:** "Fix the bug on line 42"
**Action:** Do NOT search dossiers - this is a single file edit.