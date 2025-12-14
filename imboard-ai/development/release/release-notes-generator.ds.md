---
authors:
- name: Yuval Dimnik <yuval.dimnik@gmail.com>
checksum:
  algorithm: sha256
  hash: 6b01bece5a5ab6bf1b7426c74be77bcb9507f7023235b0ec2b4f788cccb2dbe7
name: release-notes-generator
objective: Generate customer-facing release notes and internal support brief from
  git history
schema_version: 1.0.0
signature:
  algorithm: ed25519
  public_key: rwZMHabZOn44qGc9tIRVPjFsHpoB3KxbsLhoULI5Xrw=
  signature: zmOtOXWc5x4AJKoCnI6/6iys3NFsukJaGREqh5QtdhzJGI1wCWkwFND0n5dAUGadhlScNeAQXgKH3AnWj1cZAg==
  signed_by: Yuval Dimnik <yuval.dimnik@gmail.com>
  timestamp: '2025-12-14T14:42:28.693458+00:00'
status: draft
title: Release Notes Generator
version: 1.0.0
---

# Release Notes Generator

You are generating release documentation for a software release. You will produce TWO complementary artifacts:
1. **Customer-facing release notes** - What users need to know about this release
2. **Internal support brief** - What support/engineering teams need that ISN'T in the customer notes

These documents do NOT overlap. The internal brief assumes the reader has seen the customer release notes.

## Your Task

Analyze all changes since the last release tag and produce:
1. Customer-facing release notes covering features, fixes, security updates, breaking changes, deprecations, and migrations
2. Internal support brief covering known issues, technical root causes, workarounds, and things NOT communicated externally

## Context to Gather

### 1. Release Boundaries
- Identify the last release tag (ask if unclear)
- Determine the comparison range: `[last-tag]..HEAD` or `[last-tag]..[new-tag]`
- Note the release version being documented

### 2. Git History Analysis
- List all commits in the range: `git log --oneline [range]`
- Look for conventional commit prefixes: `feat:`, `fix:`, `security:`, `BREAKING CHANGE:`, `deprecate:`, `docs:`, `chore:`
- Check for merge commits that reference PR numbers

### 3. PR Context (if available)
- Extract PR titles and numbers from merge commits
- Look for labels: `breaking`, `security`, `feature`, `bug`, `deprecation`, `dependencies`
- Note any PR descriptions that explain the "why"

### 4. Dependency Changes
- Compare `package.json`, `requirements.txt`, `Cargo.toml`, or equivalent
- Note major version bumps and security-related updates
- Check for CVE fixes

### 5. Known Issues & Internal Context
- Check issue tracker for open bugs affecting this release
- Look for `TODO`, `FIXME`, `HACK`, `WORKAROUND` comments in changed files
- Note any partial fixes or known limitations mentioned in PRs
- Identify technical debt introduced or deferred

### 6. Compatibility Information
- Check for changes to minimum supported versions (Node, Python, browsers, etc.)
- Note any new runtime requirements
- Identify deprecated platform support

### 7. Changelog Conventions
- Check for existing `CHANGELOG.md` format
- Match the existing style (Keep a Changelog, conventional, custom)

---

## Output Format

Produce both documents below. They are separate and complementary - do NOT duplicate content.

---

# CUSTOMER RELEASE NOTES

> **Tone guidance:** Write in active voice, second person ("you can now..."). Lead with user benefits, not technical implementation. Be specific and scannable. Avoid jargon. Match the organization's brand voice - omit section icons if the tone is formal/enterprise.

## [Product Name] v[X.Y.Z] Release Notes

**Release Date:** [YYYY-MM-DD]

### Overview

[One paragraph: Theme of this release and 2-3 major highlights. Focus on user benefits.]

### Compatibility

| Requirement | Supported Versions |
|-------------|-------------------|
| [Runtime/Platform] | [Version range] |
| [Browser/OS] | [Version range] |

**Upgrade path:** This release supports upgrades from v[X.X.X] and later. [Any special notes.]

---

### Breaking Changes

> **Action Required:** [Yes/No]

[If Yes, include this migration guide. If No, state "No breaking changes in this release" and skip to next section.]

#### Migration Guide

**1. [Change title]**

*What changed:* [Clear explanation of the change]

*Who is affected:* [Which users/configurations are impacted]

*How to migrate:*
1. [Step-by-step instructions]
2. [Next step]
3. [Verification step]

*Example:*
```
# Before
[old code/config]

# After
[new code/config]
```

---

### Security Updates

| Severity | Description | CVE |
|----------|-------------|-----|
| [Critical/High/Medium/Low] | [What was fixed] | [CVE-XXXX-XXXXX or N/A] |

*If none: "No security updates in this release."*

---

### New Features

- **[Feature name]** — [What you can now do and why it matters]
- **[Feature name]** — [User benefit]

*If none: "No new features in this release."*

---

### Enhancements

- **[Enhancement area]** — [What improved and the benefit]

*If none: "No enhancements in this release."*

---

### Bug Fixes

- **[What was broken]** — [Now works correctly. Briefly describe the fix from user perspective.]

*If none: "No bug fixes in this release."*

---

### Deprecations

> **Future breaking changes:** The following features are deprecated and will be removed in a future release.

| Deprecated | Replacement | Removal Target |
|------------|-------------|----------------|
| [Feature/API/Config] | [What to use instead] | v[X.Y.Z] or [date] |

*If none: "No new deprecations in this release."*

---

### Documentation

- [Link to full release documentation]
- [Link to migration guide if applicable]
- [Link to API changelog if applicable]

### Questions?

[Support contact or link to support channel]

---

## Changelog Entry

```markdown
## [X.Y.Z] - YYYY-MM-DD

### Breaking Changes
- [Change with migration reference]

### Security
- [Security fix with severity]

### Added
- [New feature]

### Changed
- [Enhancement]

### Fixed
- [Bug fix]

### Deprecated
- [Deprecated feature]
```

---

# INTERNAL SUPPORT BRIEF

> **For internal use only. Complements the customer release notes - assumes reader has seen them.**

## v[X.Y.Z] Support Brief

### Known Issues NOT Disclosed Externally

| Issue | Customer Symptoms | Workaround | Tracking |
|-------|-------------------|------------|----------|
| [Description] | [What user will see/report] | [Steps support can suggest] | [Issue link] |

*If none: "No known undisclosed issues at release time."*

### Technical Root Causes

Context for bugs/security fixes listed in customer notes:

| Customer-Facing Item | Actual Root Cause | Code Changed |
|----------------------|-------------------|--------------|
| [Bug/security fix name] | [Technical explanation] | [Files/lines] |

### Anticipated Support Questions

**Q: [Question customers will likely ask about a change]**
A: [Answer with internal context support needs]

### Regression Risks

| Area | Risk | What to Monitor |
|------|------|-----------------|
| [Component/feature] | [What could go wrong] | [Metrics/logs to watch] |

### Incomplete Fixes / Deferred Work

| Issue | What Shipped | What Didn't | Expected Timeline |
|-------|--------------|-------------|-------------------|
| [Issue] | [Partial fix] | [Gap remaining] | [When or TBD] |

### Rollback Procedure

**Safe to rollback?** [Yes / No / Partial]

**Steps:**
1. [Step]
2. [Step]

**Irreversible changes:** [Data migrations, schema changes, or "None"]

**Rollback testing:** [Was rollback tested? Notes]

---

## Example Output

### Customer Release Notes Example

```markdown
## Acme Platform v2.3.0 Release Notes

**Release Date:** 2025-01-15

### Overview

This release introduces webhook notifications for real-time event updates and resolves a session stability issue affecting some users. All customers using custom OAuth configurations must update their settings before upgrading.

### Compatibility

| Requirement | Supported Versions |
|-------------|-------------------|
| Node.js | 18.x, 20.x, 22.x |
| Browsers | Chrome 90+, Firefox 88+, Safari 14+, Edge 90+ |

**Upgrade path:** Supports upgrades from v2.1.0 and later.

---

### Breaking Changes

> **Action Required:** Yes

#### Migration Guide

**1. OAuth configuration location changed**

*What changed:* OAuth settings moved from `auth.oauth` to `auth.providers.oauth` in your configuration file.

*Who is affected:* Customers using custom OAuth providers (SSO, enterprise identity).

*How to migrate:*
1. Open your `config.yaml` file
2. Locate the `auth.oauth` section
3. Move it under `auth.providers.oauth`
4. Run `acme validate-config` to verify
5. Restart your application

*Example:*
```yaml
# Before (v2.2.x)
auth:
  oauth:
    provider: okta
    client_id: xxx

# After (v2.3.0)
auth:
  providers:
    oauth:
      provider: okta
      client_id: xxx
```

---

### Security Updates

| Severity | Description | CVE |
|----------|-------------|-----|
| High | Fixed prototype pollution in data parser | CVE-2025-1234 |
| Medium | Resolved open redirect in OAuth callback | N/A |

---

### New Features

- **Webhook notifications** — You can now receive instant HTTP callbacks when orders ship, payments complete, or errors occur. Configure endpoints in Settings > Integrations.
- **Bulk export API** — Export up to 10,000 records in a single request, reducing pagination overhead for large data pulls.

---

### Bug Fixes

- **Unexpected session logouts** — Fixed an issue where sessions expired prematurely on Safari and Firefox.
- **CSV exports in Excel** — Exported files now open correctly in Excel without character encoding issues.

---

### Deprecations

| Deprecated | Replacement | Removal Target |
|------------|-------------|----------------|
| `GET /api/v1/export` | `POST /api/v2/export` | v3.0.0 (Q3 2025) |
| `config.legacy_auth` | `config.auth.providers` | v2.5.0 |

---

### Documentation

- [Full v2.3.0 documentation](https://docs.example.com/v2.3.0)
- [OAuth migration guide](https://docs.example.com/migrate-oauth)

### Questions?

Contact support@example.com or visit our [Help Center](https://help.example.com).
```

### Internal Support Brief Example

```markdown
## v2.3.0 Support Brief

### Known Issues NOT Disclosed Externally

| Issue | Customer Symptoms | Workaround | Tracking |
|-------|-------------------|------------|----------|
| Webhook retry storms | If customer endpoint responds >5s, queued webhooks flood when endpoint recovers | Pause webhooks in settings; eng can adjust retry per-account | GH-892 |
| Bulk export memory | Exports >8k records may timeout on small instance tiers | Try off-peak; escalate if <8k records | GH-901 |

### Technical Root Causes

| Customer-Facing Item | Actual Root Cause | Code Changed |
|----------------------|-------------------|--------------|
| Session logout fix | JWT refresh check used `<` instead of `<=`, causing 1-second false expiry window | `src/auth/refresh.ts:42` |
| CSV encoding fix | Missing UTF-8 BOM header that Excel requires for non-ASCII | `src/export/csv.ts:118` |
| Prototype pollution | User-supplied JSON parsed without sanitization | `src/parser/json.ts:55-70` |

### Anticipated Support Questions

**Q: Webhooks aren't firing for a customer**
A: Check: 1) URL is HTTPS, 2) webhook logs in admin panel, 3) endpoint returns 2xx within 5s. If all good, likely GH-892 retry storm—check queue depth.

**Q: Customer's export is "stuck"**
A: Check record count. If >8k, this is GH-901—suggest off-peak hours or filtered export. Escalate to eng if <8k records.

**Q: Customer asks about CVE-2025-1234 impact**
A: Exploitable only with direct API access and malformed JSON payload. No evidence of exploitation. Fixed in this release, no customer action needed.

### Regression Risks

| Area | Risk | What to Monitor |
|------|------|-----------------|
| Webhook throughput | High-volume accounts may hit rate limits at scale | `webhook_queue_depth` metric, error logs for 429s |
| OAuth migration | Customers may misconfigure during migration | Auth failure rate by provider, support ticket volume |

### Rollback Procedure

**Safe to rollback?** Yes, with caveat

**Steps:**
1. Deploy v2.2.x tag via standard deploy process
2. If customers already migrated OAuth config, run `scripts/rollback-oauth-config.sh`

**Irreversible changes:** None

**Rollback testing:** Tested in staging 2025-01-10
```

---

**Instructions:**
- Always produce BOTH documents - they are complementary, not overlapping
- **Customer notes tone:** Active voice, second person, benefit-focused, no jargon
- **Internal brief tone:** Technical, direct, honest about limitations
- Security fixes get their own section in customer notes - don't bury in bug fixes
- For breaking changes, always include a full migration guide with examples
- Deprecations warn about FUTURE removals; breaking changes are NOW
- If a section has no content, state "None in this release" - don't leave blank
- Match the organization's existing changelog format if one exists
- This is READ-ONLY analysis - do not modify any files unless explicitly asked