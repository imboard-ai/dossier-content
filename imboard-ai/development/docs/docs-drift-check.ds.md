---
authors:
- name: yuvaldim
checksum:
  algorithm: sha256
  hash: 19018fcca792ac1249dfcac28bc088e8b5c360acef663b7813fe72cd2d90dc2d
name: docs-drift-check
objective: Detect documentation drift from code and propose fixes
schema_version: 1.0.0
signature:
  algorithm: ed25519
  public_key: rwZMHabZOn44qGc9tIRVPjFsHpoB3KxbsLhoULI5Xrw=
  signature: OPKrwkNL5sZp0GTQocMLAv8aIn+GcN2glCN0Fr1J5KslUnkJai3jASIX5D1z7KRleau6gHq+1vutQ5vxmMuyAA==
  signed_by: yuvaldim
  timestamp: '2025-12-14T15:34:39.734454+00:00'
status: draft
title: Docs/README Drift Check
version: 1.0.0
---

# Docs/README Drift Check Dossier

## Task Overview

**ID:** D-004
**Title:** Docs/README ↔ code drift check
**Motivation:** Keep documentation truthful; reduce onboarding and support load
**Schedule:** Weekly; on README/docs changes; pre-release

## Objective

Detect mismatches between documentation and code (APIs, flags, environment variables, examples); identify stale sections; propose documentation patches; highlight missing documentation for new behavior.

## High-Level Workflow

1. Identify documentation files and their referenced code paths
2. Extract public API surface, CLI options, and environment variables from code
3. Compare documentation claims against actual code implementation
4. Detect stale examples and outdated instructions
5. Identify undocumented features and behavior
6. Generate drift report with specific line references
7. Propose patch snippets for fixing drift
8. Create checklist of missing documentation

## Inputs and Context

### Required Inputs

1. **Documentation files:**
   - README.md
   - All *.md files in docs/ directory
   - Inline code comments marked as public documentation
   - API documentation files

2. **Code context:**
   - Public API surface (exported functions, classes, methods)
   - CLI argument parsers and help text
   - Environment variable references
   - Configuration file schemas
   - Example code in docs/ or examples/ directories

3. **Version information:**
   - Current version from package metadata
   - Git commit hash for traceability

### Input Validation

Before proceeding, verify:
- Documentation directory exists and contains files
- Code repository is accessible
- Can locate main entry points (CLI, API modules)
- Version information is available

## Execution Process

### Step 1: Discover Documentation Files

Action: Locate all documentation files
- Find README.md in repository root
- Glob for **/*.md in docs/ directory
- Identify example code blocks in documentation
- Note any API reference files

**Output:** List of documentation file paths with line counts

### Step 2: Extract Code Truth

Action: Build ground truth from code
- Parse CLI argument definitions (argparse, click, typer, etc.)
- Extract environment variable reads (os.getenv, ENV references)
- List public API exports (classes, functions, decorators)
- Extract configuration schema (pydantic, dataclass, JSON schema)
- Capture CLI --help output if executable

**Output:** Structured data of:
- CLI flags and their types, defaults, help text
- Environment variables and their purpose
- Public API signatures with parameters and return types
- Configuration keys and validation rules

### Step 3: Parse Documentation Claims

Action: Extract assertions from documentation
- Find all code examples and snippets
- Identify documented CLI usage patterns
- Extract environment variable references
- Note documented API signatures and examples
- Identify version-specific claims

**Output:** List of documentation claims with file:line references

### Step 4: Cross-Reference and Detect Drift

Action: Compare docs against code truth

For each documentation claim:
1. Find corresponding code implementation
2. Verify signatures match (parameter names, types, defaults)
3. Check CLI flags exist and have correct behavior
4. Validate environment variables are actually used
5. Test example code syntax (static analysis only)
6. Flag deprecated features still documented
7. Note renamed or moved APIs

**Drift categories:**
- MISSING_IN_CODE: Documented feature does not exist
- SIGNATURE_MISMATCH: Parameters, types, or defaults differ
- DEPRECATED: Feature is deprecated but docs do not mention it
- RENAMED: API was renamed but docs use old name
- STALE_EXAMPLE: Example uses outdated syntax or imports
- WRONG_DEFAULT: Documented default value does not match code

### Step 5: Identify Missing Documentation

Action: Find undocumented features
- List public APIs not mentioned in docs
- Find CLI flags without documentation examples
- Identify environment variables not in docs
- Note new features added since last release

**Output:** List of undocumented items with severity (critical/nice-to-have)

### Step 6: Generate Drift Report

Create structured report with:

#### Summary Section
- Total drift issues found
- Breakdown by severity (critical/medium/low)
- Breakdown by category
- Overall documentation health score

#### Detailed Findings

For each drift issue:
- Issue number and category with brief description
- Location: docs/file.md:line
- Current docs say: exact quote
- Code reality: actual signature/behavior
- Severity: critical/medium/low
- Suggested fix: patch snippet or description

#### Missing Documentation Checklist

List of features/variables/flags needing documentation

### Step 7: Generate Patch Snippets

For each fixable drift issue, provide unified diff format patches, one patch per file.

## Output Artifacts

### 1. Drift Report (drift-report.md)

Markdown file containing:
- Executive summary
- Detailed findings with line references
- Severity breakdown
- Recommended actions

### 2. Suggested Patches (drift-patches/)

Directory containing:
- One .patch or .diff file per documentation file
- PR-ready diffs that can be reviewed and applied
- Each patch annotated with issue number

### 3. Missing Documentation Checklist (missing-docs.md)

Structured checklist:
- Critical missing docs (new public APIs)
- Important missing docs (CLI flags, env vars)
- Nice-to-have docs (internal details, advanced usage)

### 4. Machine-Readable Report (drift-report.json)

JSON structure for automation with timestamp, version, commit, summary stats, issues array, and missing docs list.

## Safety Constraints and Boundaries

### Read-Only Operations
- NEVER modify documentation files automatically
- NEVER commit or push changes without explicit approval
- ONLY generate suggested patches for human review

### Evidence-Based Claims
- DO NOT infer behavior without code evidence
- DO NOT assume runtime behavior from static analysis alone
- CLEARLY mark speculative suggestions as such
- Reference specific code locations for all claims

### Scope Limitations
- Focus on public API and user-facing documentation
- Ignore internal implementation docs unless explicitly included
- Do not validate correctness of algorithms, only API contracts
- Do not test runtime behavior (no code execution)

### Error Handling
- If code parsing fails, report specific files and continue
- If documentation is unparseable, flag for manual review
- Do not fail entire check due to one problematic file
- Log all errors to separate error log file

## Success Criteria

A successful drift check run should:
- Analyze all documentation files without crashing
- Generate actionable drift report with specific line numbers
- Provide patch snippets that are syntactically correct
- Identify at least critical missing documentation
- Complete within reasonable time (less than 5 minutes for typical repo)
- Produce both human-readable and machine-readable outputs

## Example Usage Scenarios

### Scenario 1: New CLI flag added but README not updated

**Code change:** Added --verbose flag to CLI parser
**Documentation:** README shows only --quiet flag
**Expected detection:** Issue flagged as MISSING_IN_DOCS with medium severity

### Scenario 2: API signature changed

**Code change:** Function process(data) now requires process(data, config=None)
**Documentation:** API docs show process(data)
**Expected detection:** Issue flagged as SIGNATURE_MISMATCH with critical severity

## Automation Integration

### Pre-Commit Hook
Run on README or docs/** changes. If critical issues found, block commit with warning.

### Weekly CI Job
Schedule: weekly. Steps: checkout code, run drift check, upload artifacts, create issue if critical drift found.

### Pre-Release Gate
Before tagging release, run drift check. If any critical issues, fail release pipeline and require manual review.

## Notes and Considerations

- Language-specific parsers: May need different strategies for Python, JS, Go, etc.
- Documentation style: Adapt to project doc style (API reference vs tutorials)
- False positives: Some drift may be intentional (simplified examples); allow ignore rules
- Ignore patterns: Support .driftignore file for known false positives
- Incremental checks: Option to check only changed docs for faster feedback

## Version History

- v1.0: Initial dossier created
- Scope: Documentation drift detection and reporting
- Target: Multi-language repositories with markdown documentation