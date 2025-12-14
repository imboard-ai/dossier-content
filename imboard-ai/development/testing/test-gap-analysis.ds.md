---
authors:
- name: yuvaldim
checksum:
  algorithm: sha256
  hash: 6d7c421a132b212f01eca2c812e386fdc8714a9c5b99f78f3c2b88828f75c9ee
name: test-gap-analysis
objective: Analyze test coverage gaps and suggest high-leverage test cases
schema_version: 1.0.0
signature:
  algorithm: ed25519
  public_key: rwZMHabZOn44qGc9tIRVPjFsHpoB3KxbsLhoULI5Xrw=
  signature: p0+1WtsAo7OwHDPAuXxijMCe8IPMuNHnQ4P1IWcqnrDIw4oL0m7I4tH4AFWm5U3Xd+8owQ5YVQRRjWYAJPlSAA==
  signed_by: yuvaldim
  timestamp: '2025-12-14T15:36:31.984777+00:00'
status: draft
title: Test Gap Analysis
version: 1.0.0
---

# Test Gap Analysis & Suggested Tests

## Overview

This dossier defines a systematic workflow for identifying test coverage gaps and proposing high-leverage test cases to reduce regressions and improve coverage where it matters most.

## Metadata

- **Task ID**: D-009
- **Title**: Test gap analysis & suggested tests
- **Frequency**: Monthly; pre-release hardening; after bug spike
- **Mode**: Read-only analysis

## Motivation

Maintain robust test coverage by:
- Reducing regressions in critical code paths
- Improving coverage in high-risk modules
- Prioritizing testing effort where it delivers maximum value
- Avoiding low-value, brittle tests

## Inputs & Context

### Required Inputs

1. **Test Folders**
   - Locate all test directories (e.g., \`tests/\`, \`__tests__/\`, \`*.test.js\`, \`*.spec.py\`)
   - Catalog existing test files and their targets

2. **Coverage Reports** (if available)
   - Parse coverage data (e.g., \`coverage.xml\`, \`.coverage\`, \`lcov.info\`)
   - Identify uncovered or poorly covered modules
   - Note: If unavailable, infer from test/source file ratios

3. **Recent Changes**
   - Review recent PRs or git commits (last 1-3 months)
   - Identify frequently modified files
   - Flag modules with changes but no corresponding test updates

4. **Module Ownership & Architecture**
   - Understand critical paths (e.g., auth, data processing, API endpoints)
   - Identify core vs. peripheral modules
   - Review dependency graphs if available

### Validation

Before proceeding, verify:
- At least one test directory exists
- Source code directory is accessible
- Git history is available (for change analysis)

## Workflow

### Step 1: Discover & Catalog Tests

**Actions:**
1. Use file search to locate all test files
2. Map tests to their corresponding source files
3. Identify untested source files
4. Calculate basic test-to-source ratios per module

**Output:** Test inventory with mappings

### Step 2: Analyze Coverage

**Actions:**
1. If coverage report exists:
   - Parse coverage percentages per file/module
   - Identify files with <50% coverage
   - Find critical files (by name/path) with <80% coverage
2. If no coverage report:
   - Estimate coverage by test file presence
   - Flag source files without corresponding tests

**Output:** Coverage analysis with highlighted gaps

### Step 3: Identify Critical & Risky Modules

**Actions:**
1. Classify modules by criticality:
   - **Critical**: Auth, payment, data integrity, security, core APIs
   - **High-risk**: Complex logic, recent bug fixes, frequent changes
   - **Standard**: Business logic, utilities, helpers
   - **Low-priority**: UI components, formatting, deprecated code

2. Analyze git history:
   - Files changed most frequently (top 20%)
   - Files involved in recent bug fixes
   - Large or complex files (>500 lines) without tests

**Output:** Risk-scored module list

### Step 4: Evaluate Existing Tests

**Actions:**
1. Scan existing tests for quality indicators:
   - Integration vs. unit test balance
   - Brittle patterns (snapshot overuse, hard-coded values)
   - Test coverage of edge cases
   - Assertion density and meaningfulness

2. Identify test smells:
   - Flaky tests (if CI logs available)
   - Tests without assertions
   - Overly broad or narrow tests

**Output:** Test quality assessment

### Step 5: Propose Missing Test Cases

**Actions:**
1. For each high-priority gap, suggest:
   - Test type (unit, integration, e2e)
   - Specific scenarios to cover
   - Edge cases and error conditions
   - Input validation tests

2. Prioritize suggestions by:
   - **P0**: Critical uncovered paths (auth, security, data loss scenarios)
   - **P1**: High-risk modules with <50% coverage
   - **P2**: Frequently changed files without recent test updates
   - **P3**: Nice-to-have coverage improvements

**Output:** Prioritized test case list

### Step 6: Generate Test Scaffolds

**Actions:**
1. For top 5-10 priority gaps, provide:
   - High-level test structure (not full implementation)
   - Example test case outline
   - Suggested test framework/patterns
   - Mock/fixture requirements

2. Keep scaffolds minimal:
   - Focus on structure, not exhaustive cases
   - Avoid generating >100 lines per scaffold
   - Provide patterns, not copy-paste boilerplate

**Output:** Test scaffold examples

## Output Artifact

### Coverage & Gap Report

**Format:** Markdown document containing:

\`\`\`markdown
# Test Coverage Gap Analysis Report

**Generated**: [timestamp]
**Repository**: [repo name]
**Analysis Period**: [date range]

## Executive Summary
- Total source files: X
- Files with tests: Y (Z%)
- Critical files tested: A/B (C%)
- High-priority gaps: N

## Coverage Overview
[Table or summary of coverage by module]

## Risk-Scored Gaps
### Priority 0: Critical Gaps
- [Module/File]: [reason] - Coverage: X%
  - Risk factors: [frequent changes / security / data integrity]
  - Suggested tests: [high-level scenarios]

### Priority 1: High-Risk Gaps
[...]

### Priority 2: Medium-Priority Gaps
[...]

## Test Quality Observations
- [Findings about existing test patterns]
- [Identified test smells or improvements]

## Recommended Actions
1. [Most important test to write]
2. [Second priority]
[...]

## Test Scaffolds
### [Module/Feature Name]
\`\`\`language
// High-level test structure
describe('[Feature]', () => {
  it('[scenario]', () => {
    // Arrange
    // Act
    // Assert
  });
});
\`\`\`

[Additional scaffolds for top priorities]
\`\`\`

### Prioritized Test Case List

**Format:** CSV or markdown table:

| Priority | Module/File | Scenario | Test Type | Rationale | Effort |
|----------|-------------|----------|-----------|-----------|--------|
| P0 | auth/login.js | Invalid token handling | Unit | Security critical, no coverage | M |
| P0 | payments/process.js | Double-charge prevention | Integration | Data integrity, recent bugs | H |
| P1 | api/users.js | Input validation | Unit | Frequent changes, 30% coverage | L |
[...]

## Safety Constraints & Boundaries

### What This Workflow DOES:
- Analyze existing code and tests (read-only)
- Identify gaps based on heuristics and data
- Propose test cases and priorities
- Provide minimal scaffolds as examples

### What This Workflow DOES NOT Do:
- Modify any source or test files
- Generate complete, runnable test suites
- Create hundreds of snapshot tests
- Execute tests or measure actual coverage (relies on reports)
- Make code changes to improve testability

### Quality Guardrails:
1. **Focus on high-leverage tests**: Prioritize critical paths over exhaustive coverage
2. **Avoid brittle tests**: Don't suggest excessive snapshot or UI tests
3. **Keep suggestions actionable**: Limit to top 20-30 test cases
4. **Minimal scaffolds**: Provide patterns, not full implementations
5. **Read-only mode**: Never modify existing files

### Scope Limitations:
- Analysis limited to accessible files and provided coverage data
- Risk scoring is heuristic-based (not absolute)
- Suggestions are guidance, not requirements
- Manual review required before implementing tests

## Success Criteria

A successful execution produces:
1. Clear, actionable gap analysis
2. Prioritized list of 10-30 high-value test cases
3. 3-10 test scaffold examples
4. Report that takes <15 minutes to review
5. Recommendations that can be implemented incrementally

## Notes & Considerations

- **Coverage metrics**: 100% coverage is not the goal; focus on critical paths
- **Test pyramid**: Ensure suggestions maintain healthy unit/integration/e2e balance
- **Maintenance burden**: Avoid suggesting tests that will become obsolete quickly
- **Context matters**: Use module names and code structure to infer criticality
- **Incremental improvement**: Prioritize a few high-value tests over many low-value ones

## Version History

- **v1.0**: Initial dossier (D-009)