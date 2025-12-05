---
authors:
- name: tal@liberio.ai
checksum:
  algorithm: sha256
  hash: d824d6d9779e02eb6e50c21144c51a38a97c611f9dab518ee30eb6cad3470bc0
name: dependency-capability-check
objective: Check if a project's dependencies support a specific feature or version
  requirement
schema_version: 1.0.0
signature:
  algorithm: ed25519
  public_key: aFl367Ta9sFHpQZXSsndkGYm4h4BgSuCV/f/qkMIvl4=
  signature: DypxBRsCA4ICYDYkwZfm48eUUhr3ZE550AZG3dFuCRWwjiNU8wzHtunnPda7c5vqZWSa8eDW1rdVbBBdhBNVAg==
  signed_by: tal@liberio.ai
  timestamp: '2025-12-05T08:09:55.131680+00:00'
status: draft
title: Dependency Capability Check
version: 1.0.0
---

# Dependency Capability Check

Check if the project's installed dependencies support a specific feature.

## Your Task

Determine if: **{capability}**

### 1. Identify Current Versions

Check the project's dependency files:
- `package.json` / `package-lock.json` (Node.js)
- `requirements.txt` / `pyproject.toml` (Python)
- `go.mod` (Go)
- `Cargo.toml` (Rust)
- `Gemfile` / `Gemfile.lock` (Ruby)

Find the installed version of the relevant package.

### 2. Research Feature Availability

Check when the feature was introduced:
- Package changelog or release notes
- GitHub releases page
- Documentation version history
- Migration guides

### 3. Compare Versions

Determine if current version >= required version for the feature.

### 4. Check for Breaking Changes

If upgrade is needed:
- Are there breaking changes between current and required version?
- What migration steps are needed?
- Are there peer dependency conflicts?

## Output Format

### Answer
**[YES/NO]**: {capability}

### Current State

| Package | Current Version | Required Version | Status |
|---------|-----------------|------------------|--------|
| {name} | {current} | {required} | Supported / Upgrade needed |

### Evidence

**Feature introduced in**: v{version} ({date if known})

**Source**: {link to changelog, docs, or release notes}

**Current usage in codebase** (if any):
- `path/to/file.ts:45` - {how it's currently used}

### Upgrade Path (if needed)

**From** {current} **to** {required}:

1. **Breaking changes to address**:
   - {change 1}
   - {change 2}

2. **Peer dependency updates**:
   - {peer} needs to be updated to {version}

3. **Migration steps**:
   ```bash
   npm install {package}@{version}
   ```

4. **Code changes required**:
   - {file}: {what to change}

### Alternatives (if upgrade not feasible)

- {Alternative approach that works with current version}
- {Polyfill or shim option}

## Notes

- Always check lock files for actual installed versions, not just ranges
- Consider transitive dependencies that might conflict
- Check if the feature requires additional peer dependencies
- Look for canary/beta versions if feature is very new