---
authors:
- name: Yuval Dimnik
checksum:
  algorithm: sha256
  hash: b16a6743beab45b74cf49cbfdbbdd8ca7988752da294637694533e99d03a1566
description: 'Run `dossier list --json` at START of workflow tasks (setup, deploy,
  migrate, refactor, CI/CD). Once per conversation. Output ''Dossier check: [result]''
  after running. Skip if you see this marker in conversation history.'
name: dossier-discovery
objective: Find and suggest relevant dossier workflows for complex tasks
schema_version: 1.0.0
status: draft
title: Dossier Discovery
version: 1.1.0
---

# Dossier Discovery

## DECISION TREE

```
Is this a workflow task (setup, deploy, migrate, refactor, CI/CD)?
|-- NO --> Skip this skill
+-- YES --> Is "Dossier check:" visible in conversation history above?
            |-- YES --> Skip, already checked
            +-- NO --> Run this command NOW:
                       dossier list --json
```

## After Running Command

**MUST output this marker** (enables tracking for rest of conversation):

```
Dossier check: [X matching dossiers found] or [no matches]
```

If matches found, list them and ask user which to run.
If no matches, state this and proceed with the task.

---

## Matching Workflow Tasks

Trigger this skill for:
- Setup/configuration ("set up X", "configure Y", "create workflow")
- Deployment/migration ("deploy", "migrate", "release")
- CI/CD ("add pipeline", "set up tests", "automate")
- Refactoring ("refactor X", "restructure", "reorganize")
- Multi-step workflows ("workflow for X", "process for Y")

## NOT Workflow Tasks (skip)

Do NOT trigger for:
- Questions ("what does X do?", "how does Y work?")
- Single file edits ("fix typo", "update line 42")
- Quick lookups ("find X", "where is Y defined")
- Explicit dossier commands (user already running a dossier)

---

## Handling Results

### Good Match Found

If a dossier closely matches the user's intent:

```
Dossier check: 1 matching dossier found

Found: {name}
"{title}"

Run it? (y/n)
```

If user says yes, run:
```bash
dossier run {name}
```

### Multiple Matches Found

If several dossiers might be relevant:

```
Dossier check: {count} matching dossiers found

1. {name1} - "{title1}"
2. {name2} - "{title2}"
3. {name3} - "{title3}"

Which would you like to run? (1-{count}, or 'skip' to proceed manually)
```

### No Match Found

```
Dossier check: no matches
```

Proceed with the task normally. After completing a complex workflow, optionally suggest:

```
This workflow might be useful to save as a dossier for future use.
Would you like to create one? (y/n)
```

If yes, run:
```bash
dossier run imboard-ai/meta/create-dossier
```

---

## Search Priority

When matching dossiers to tasks:

1. **Exact keyword matches** in title or description
2. **Category matches** (e.g., "react" task matches "react" category)
3. **Semantic similarity** to objective/description

## Examples

**User:** "Help me set up a new React component library"
**Action:** Run `dossier list --json`, find "imboard-ai/development/setup-react-library", output marker, suggest running it.

**User:** "What's in the package.json?"
**Action:** Skip - this is a simple lookup, not a workflow task.

**User:** "Create a workflow to sync my git worktrees"
**Action:** Run `dossier list --json`, find git-related dossiers, output marker, suggest relevant ones.

**User:** "Fix the bug on line 42"
**Action:** Skip - this is a single file edit, not a workflow task.