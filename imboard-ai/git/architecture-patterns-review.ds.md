---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "architecture-patterns-review",
  "title": "Architecture & Pattern Consistency",
  "version": "1.1.0",
  "protocol_version": "1.0",
  "status": "Draft",
  "objective": "Identify inconsistent patterns, duplicate approaches, and architectural drift in the codebase",
  "category": [
    "development"
  ],
  "risk_level": "low",
  "requires_approval": false,
  "authors": [
    {
      "name": "Yuval Dimnik <yuval.dimnik@gmail.com>"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "c59dd25df423be177ffe09312b433167587d232242ac9de02d3c70d870273e9c"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "QLn5Rdy3clb8ClvAVUVZk2iyrGoODY0aX8wT77e/uG8SXJSDXTRgu2Z2wq0LtDAvpXlO9p1mxQidWa7HWRMFAg==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-28T06:56:02.438Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Architecture & Pattern Consistency Analysis

You are analyzing the codebase for architectural consistency and pattern usage.

## Your Task

Identify how the codebase handles common concerns and whether it does so consistently.


## Scope

**In scope:** Source files in the repository, read for the patterns listed below.
**Out of scope:** Dependencies and vendored code, generated output, and any judgement about whether a pattern is fashionable — only whether the codebase applies it consistently.
**Stop when:** Every concern below has been checked across the codebase, or you have stated which you could not assess and why.

Report findings only — do not refactor, reformat, or "fix" the inconsistencies you find.

## Patterns to Investigate

### 1. Error Handling
- try/catch blocks?
- Error classes/types?
- Result types (Ok/Err pattern)?
- Error boundaries (if applicable)?
- How many different approaches exist?

### 2. Data Fetching / External Calls
- Direct fetch/axios?
- Wrapper utilities?
- Libraries (React Query, SWR, etc.)?
- Where is retry logic? Timeout handling?

### 3. State Management (if applicable)
- Local component state?
- Context?
- Global store (Redux, Zustand, etc.)?
- Multiple approaches for similar needs?

### 4. Configuration Management
- Environment variables?
- Config files (JSON, YAML, .env)?
- Where are configs located? (multiple places?)
- Hardcoded values scattered in code?

### 5. Testing Patterns
- Unit test style (describe/it, test())?
- Mocking approach?
- Test utilities location and usage?
- Coverage of different code areas?

### 6. File/Module Organization
- Grouping by feature? By type?
- Consistent across codebase?
- Import patterns (absolute, relative)?

### 7. Code Reuse
- Same logic implemented multiple times?
- Similar functions with slight variations?
- Opportunities for abstraction?

## Output Format

### Consistency Score
**Overall: [High/Medium/Low]**

[One paragraph: general architectural health, main consistency issues]

### Pattern Inventory
[List the different approaches found for each concern]

**Error Handling:**
- Approach A: try/catch (files: `x.ts:10`, `y.ts:45`)
- Approach B: Result types (files: `z.ts:100`)
- Approach C: Throw and don't catch (files: `w.ts:200`)

**[Other patterns...]**

### Inconsistencies Found
[Flag patterns where multiple approaches exist for the same concern]

1. **Error Handling Inconsistency (Critical)**
   - 3 different approaches across codebase
   - Evidence:
     - `auth.ts:45` - uses try/catch with custom errors
     - `api.ts:120` - throws strings (!!)
     - `db.ts:89` - returns error codes
   - Impact: Hard to handle errors consistently, new contributors confused
   - Recommendation: Standardize on [suggested approach]

### Duplication Hotspots
[Identify similar/duplicate code]

1. **Date Formatting Logic (3 locations)**
   - `utils/format.ts:10`
   - `components/DateDisplay.tsx:34`
   - `services/reports.ts:156`
   - Recommendation: Extract to shared utility

### Recommendations

**High Priority:**
1. [Most critical consistency fix]
   - Why: [impact]
   - Where: [files affected]
   - Effort: [estimate]

**Medium Priority:**
2. [Second priority]

**Low Priority / Future:**
3. [Nice-to-have consistency improvements]


## Section Sizing

Give each part of the report the space it earns:

- **Summary / assessment** — one paragraph, 3-5 sentences.
- **Findings** — one entry per real finding, no cap. Report everything meeting the bar above; do not select a "top N" and drop the rest.
- **Recommendations** — 3-5 items, highest impact first.

Length is not evidence of thoroughness. A finding with a `file:line` and a concrete consequence beats a paragraph of context.

---

**Instructions:**
- Focus on patterns that affect maintainability and onboarding
- Don't nitpick style (unless it causes bugs) - focus on architectural concerns
- Provide specific file:line references for each pattern
- Be pragmatic: some inconsistency is okay if it's intentional
- If architecture is clean and consistent, highlight what's done well!