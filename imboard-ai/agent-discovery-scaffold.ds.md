---dossier
{
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "name": "agent-discovery-scaffold",
  "description": "Analyze a project and generate structured entry-point documentation (AGENTS.md + module catalog) so external AI agents can understand capabilities without full codebase exploration",
  "objective": "Make any project instantly discoverable by external AI agents by generating structured manifests of capabilities, reusable modules, and entry points",
  "dossier_schema_version": "1.0.0",
  "status": "Draft",
  "title": "Agent Discovery Scaffold",
  "version": "1.1.0",
  "category": [
    "documentation",
    "development"
  ],
  "tags": [
    "agent-discovery",
    "documentation",
    "agents-md",
    "module-catalog",
    "onboarding",
    "reusability"
  ],
  "tools_required": [
    {
      "name": "node",
      "check_command": "node --version"
    },
    {
      "name": "git",
      "check_command": "git --version"
    }
  ],
  "risk_level": "low",
  "requires_approval": false,
  "risk_factors": [
    "creates_files"
  ],
  "destructive_operations": [
    "Creates or overwrites AGENTS.md in project root",
    "Creates docs/modules/ directory with module catalog files"
  ],
  "estimated_duration": {
    "min_minutes": 3,
    "max_minutes": 10
  },
  "inputs": {
    "required": [
      {
        "name": "project_dir",
        "description": "Absolute path to the project root directory",
        "type": "string",
        "example": "/home/user/projects/my-project"
      }
    ],
    "optional": [
      {
        "name": "output_format",
        "description": "Output format: 'agents-md' (single AGENTS.md), 'catalog' (AGENTS.md + per-module docs), 'both' (default)",
        "type": "string",
        "default": "both",
        "example": "agents-md"
      },
      {
        "name": "existing_claude_md",
        "description": "If true, incorporate existing CLAUDE.md content rather than duplicating it",
        "type": "boolean",
        "default": true
      }
    ]
  },
  "outputs": {
    "files": [
      {
        "path": "AGENTS.md",
        "description": "Structured project manifest for external AI agent consumption"
      },
      {
        "path": "docs/modules/README.md",
        "description": "Module catalog index with reusability assessment"
      }
    ]
  }
}
---

# Agent Discovery Scaffold

## Objective

Analyze a project's codebase and generate structured documentation that enables external AI agents to understand the project's capabilities, reusable modules, and entry points — without requiring full codebase exploration.

This is NOT about general documentation. It is specifically about **machine-readable project manifests** optimized for AI agent consumption.

## Key Distinction: CLAUDE.md vs AGENTS.md

- **CLAUDE.md**: Instructions for an agent *working inside* this project (conventions, commands, architecture details)
- **AGENTS.md**: A structured manifest for an agent *evaluating* this project from outside (what does it do, what can I reuse, how do I invoke it)

If a CLAUDE.md already exists, AGENTS.md should **reference** it for internal details rather than duplicating content.

## Prerequisites

- [ ] Project directory exists and contains source code
- [ ] Git repository initialized (for history analysis)
- [ ] Can identify the project's language/framework (package.json, pyproject.toml, go.mod, etc.)

## Execution Process

### Step 1: Project Survey

Action: Gather project metadata and structure

1. Read project manifest (package.json, pyproject.toml, Cargo.toml, go.mod, etc.)
2. Identify project name, description, version, dependencies
3. Map top-level directory structure (1 level deep, skip node_modules/vendor/dist)
4. Count source files by type
5. Check for existing documentation (README.md, CLAUDE.md, AGENTS.md, docs/)
6. Read git log for recent activity patterns (last 20 commits)

**Output:** Project metadata object with name, language, framework, file counts, existing docs list

### Step 2: Capability Discovery

Action: Identify the project's main capabilities

1. **Entry points**: Find all executable entry points
   - npm scripts (package.json scripts section)
   - CLI binaries (bin field, shebang files)
   - Main/index files
   - GitHub Actions workflows
   - Makefile targets

2. **Public API surface**: Identify exported modules/functions
   - Look for index.js/ts barrel exports
   - Find files with multiple named exports
   - Identify configuration schemas

3. **Core capabilities**: Read key files to understand what the project *does*
   - Start from README/CLAUDE.md if they exist
   - Read main entry points
   - Identify the project's primary "verbs" (generates, transforms, deploys, etc.)

**Output:** List of capabilities, each with: name, description, invocation method, input/output

### Step 3: Reusable Module Identification

Action: Find modules that could be extracted or consumed by external agents

**IMPORTANT: Discover module paths from code, not from assumptions.** Do not rely on directory names or hints from existing docs — glob and grep for actual source files, read their exports, and verify paths exist before cataloging them.

For each significant source directory/file, assess:

1. **Self-containment**: Does it have minimal internal dependencies?
2. **Clear interface**: Does it export functions with understandable signatures?
3. **General utility**: Could another project use this, or is it too domain-specific?
4. **Configuration**: Does it accept config/options, or is behavior hardcoded?

**Reusability scoring:**
- **High** (3/4+ criteria): Standalone utility, clear API, configurable, general purpose
- **Medium** (2/4): Useful but needs some adaptation
- **Low** (1/4): Tightly coupled, project-specific

**It is normal and expected for domain-specific projects to have zero High-reusability modules.** Do not inflate scores to fill a quota. A project where everything is Medium or Low is an honest assessment, not a failure.

**Export style matters for reusability:**
- Modules with **programmatic exports** (named functions/classes) are more reusable than CLI-only scripts
- For CLI-only modules (no programmatic exports), note this explicitly and suggest whether wrapping key logic in exported functions would meaningfully increase reusability
- A module that is only callable via `node script.js --flag` is harder for an external agent to integrate than one that exports `doThing(options)`

For each reusable module, extract:
- Module path and name (verified to exist on disk)
- What it does (one sentence)
- Key exports (function signatures) — or note "CLI-only" if no programmatic exports
- Dependencies (internal and external)
- Configuration options
- Example usage (from existing code or tests)

**Output:** Module catalog with reusability scores

### Step 4: Dependency and Integration Map

Action: Understand how this project connects to external systems

1. **External services**: APIs, databases, cloud services referenced in code
2. **Sibling projects**: References to other repos or directories (e.g., `../other-project`)
3. **Environment requirements**: Required env vars, credentials, API keys
4. **Data flows**: What data goes in, what comes out, where does it flow

**Output:** Integration map showing external touchpoints

### Step 5: Generate AGENTS.md

Action: Create the structured manifest

The AGENTS.md file MUST follow this exact structure:

```markdown
# {project_name}

> {one-line description}

## Quick Assessment

| Attribute | Value |
|-----------|-------|
| Language | {language} |
| Framework | {framework or "none"} |
| Type | {CLI tool / library / service / pipeline / etc.} |
| Status | {active / maintenance / archived} |
| Entry point | {main entry command or file} |

## What This Project Does

{2-3 sentence plain-English description of the project's purpose and primary workflow.
Focus on WHAT it produces, not HOW it works internally.}

## Capabilities

{Numbered list of things this project can do, each with invocation method}

### 1. {Capability Name}
- **What**: {one sentence}
- **Invoke**: `{command or function call}`
- **Input**: {what it needs}
- **Output**: {what it produces}

### 2. {Capability Name}
...

## Reusable Modules

{Only modules scored Medium or High reusability. It is normal for domain-specific projects to have no High-rated modules.}
{Include Low-rated modules only in the module catalog, not here.}

### {module_name} — {reusability_score}
- **Path**: `{relative/path}`
- **What**: {one sentence}
- **Key exports**: `{function1}`, `{function2}` {or "CLI-only (no programmatic exports)"}
- **Dependencies**: {external deps only}
- **Limitation**: {one sentence on what would need changing for reuse}
- **Usage example**:
  ```{language}
  {minimal usage example}
  ```

## Entry Points

| Command | Purpose | Safe to run? |
|---------|---------|-------------|
| `{cmd}` | {what it does} | {yes/dry-run/destructive} |

## Environment & Configuration

| Variable | Required | Purpose |
|----------|----------|---------|
| `{VAR}` | {yes/no} | {what it's for} |

## Integration Points

- **Reads from**: {data sources}
- **Writes to**: {output destinations}
- **Sibling projects**: {related repos/directories}

## For Deeper Understanding

{If CLAUDE.md exists}: See [CLAUDE.md](./CLAUDE.md) for internal architecture, conventions, and development instructions.
{If other docs exist}: See [docs/](./docs/) for {what's there}.
```

### Step 6: Generate Module Catalog (if output_format includes 'catalog')

Action: Create per-module documentation in `docs/modules/`

For each Medium/High reusability module, create `docs/modules/{module-name}.md`.
Also create docs for Low-rated modules — they belong in the catalog for reference, just not in AGENTS.md's summary.

```markdown
# {Module Name}

> {one-line description}

## Reusability: {High/Medium/Low}

## Interface

### Exports

{List each export with signature and description.
If CLI-only: note "No programmatic exports — invoked via CLI only" and list the CLI flags/args instead.
Suggest whether wrapping key logic as exports would meaningfully increase reusability.}

### Configuration

{Options/config the module accepts}

### Dependencies

| Package | Why |
|---------|-----|
| {dep} | {purpose} |

## Usage Example

```{language}
{Copy-pasteable example — import-based if exports exist, CLI invocation if CLI-only}
```

## What Would Need Changing for Reuse

{Concrete list of what an external consumer would need to adapt.
Be specific: "replace HUB_PAGES constant with your own hub definitions" not "configuration changes needed".}

1. {Specific thing to change and why}
2. {Specific thing to change and why}
3. {Any files to copy}
4. {Dependencies to install}
```

Create `docs/modules/README.md` as an index:

```markdown
# Module Catalog

Reusable modules from {project_name}.

| Module | Reusability | Description |
|--------|-------------|-------------|
| [{name}](./{name}.md) | {score} | {description} |
```

### Step 7: Validate Output

Action: Verify the generated documentation

1. All referenced file paths exist
2. All referenced commands are valid (check against package.json scripts, Makefile, etc.)
3. All referenced environment variables appear in code
4. Module export lists match actual exports
5. No sensitive information (API keys, passwords) included

If validation finds issues, fix them before writing files.

### Step 8: Write Files

Action: Write the generated files to disk

1. Write `AGENTS.md` to project root
2. If catalog output: create `docs/modules/` directory, write module files and index
3. Report what was created

## Safety Constraints

- NEVER modify existing source code
- NEVER modify CLAUDE.md (it serves a different purpose)
- If AGENTS.md already exists, show diff and ask before overwriting
- Do not include secrets, credentials, or internal URLs in output
- Module reusability assessments must be evidence-based (cite specific code)

## Success Criteria

- [ ] AGENTS.md is parseable by an AI agent in under 30 seconds
- [ ] An external agent reading only AGENTS.md can answer: "What does this project do?"
- [ ] An external agent can identify reusable modules without reading source code
- [ ] All entry point commands listed are valid and runnable
- [ ] No duplication with existing CLAUDE.md content
- [ ] Module reusability scores are justified

## Anti-Patterns to Avoid

- **Don't dump the whole architecture**: AGENTS.md is a menu, not a textbook
- **Don't list every file**: Only highlight entry points and reusable modules
- **Don't duplicate CLAUDE.md**: Reference it, don't copy it
- **Don't overstate reusability**: If a module needs heavy adaptation, score it Low and skip it
- **Don't include internal jargon**: An external agent has zero context — use plain English
