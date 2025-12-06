---
authors:
- name: Yuval Dimnik <yuval.dimnik@gmail.com>
checksum:
  algorithm: sha256
  hash: 0d93e13c2b3a6dd2a8a944fc6f468500c446dbf99cc68cdb738d5bde52591118
name: readme-reality-check
objective: Compare what the README promises versus what's actually implemented in
  the codebase to identify documentation drift and hidden features
schema_version: 1.0.0
signature:
  algorithm: ed25519
  public_key: rwZMHabZOn44qGc9tIRVPjFsHpoB3KxbsLhoULI5Xrw=
  signature: /eVXrQ+NTMSlwDWxADJWWzRWngpUGfqLpKxAlPsCwnG8b4SR9wE4C/IEUNOa9wIouO3Qr2Y8qVH5l2Ox1H1VBg==
  signed_by: Yuval Dimnik <yuval.dimnik@gmail.com>
  timestamp: '2025-12-06T14:30:19.906418+00:00'
status: draft
title: README Reality Check
version: 1.0.0
---

# README Reality Check

You are analyzing the gap between documentation promises and actual implementation.

## Your Task

1. **Read the README**
   - Extract all claimed features, capabilities, and instructions
   - Note any setup steps, CLI commands, API examples
   - Identify version numbers and compatibility claims

2. **Verify Against Code**
   - Check if claimed features actually exist
   - Verify CLI commands/flags work as documented
   - Validate example code actually runs
   - Check if setup instructions are complete and accurate

3. **Find Hidden Gems**
   - Identify features that exist in code but aren't documented
   - Look for useful utilities, flags, or capabilities not mentioned

4. **Assess Impact**
   - Which gaps are critical (block new users)?
   - Which are minor (cosmetic inconsistencies)?
   - Which undocumented features should be highlighted?

## Output Format

### Executive Summary
[One paragraph: overall state of README accuracy, biggest gaps, confidence level for new users]

### Promises Kept ✅
- Feature X works as documented (verified in `file.ts:123`)
- Setup step Y is accurate (tested in `setup.sh:45`)
[Focus on 3-5 most important verified claims]

### Promises Broken ❌
- **Critical**: Claim X in README but code shows Y
  - README says: [quote with line number]
  - Reality: [evidence from code with file:line]
  - Impact: [who this affects and how]

### Undocumented Features 🎁
- Feature X exists but not mentioned (see `feature.ts:200`)
  - What it does: [brief description]
  - Why it's useful: [user value]

### Recommendations
1. [Most critical fix needed]
2. [Second priority]
3. [Nice-to-have improvements]

---

**Instructions:**
- Be specific with file paths and line numbers
- Quote exact text from README when showing discrepancies
- Focus on impact to users (especially new contributors)
- Don't nitpick minor wording - focus on functional gaps
- If README is accurate, celebrate that! (rare and valuable)