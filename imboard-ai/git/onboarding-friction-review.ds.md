---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "onboarding-friction-review",
  "title": "Onboarding Friction Assessment",
  "version": "1.1.0",
  "protocol_version": "1.0",
  "status": "Draft",
  "objective": "Identify pain points and confusion points for new contributors trying to understand and work with the project",
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
    "hash": "4463b197f53e8ef7562c10cd297dc930c28394825f78a9497dd95fe07a14e744"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "4uOT0g13s7ETD89b1TWdq8GoJT+YjTmSI041NXoQCKn8LxNbDJBKChXnIdvQ39LRmPj5ab/YYtZUyNETKWrFCQ==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-28T06:56:08.194Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Onboarding Friction Assessment

You are evaluating the new contributor experience by identifying friction points.

## Your Task

Simulate the journey of a new contributor who wants to:
1. Understand what this project does
2. Set up a development environment
3. Understand the codebase structure
4. Make their first contribution
5. Run tests and verify their changes

For each stage, identify friction points.


## Scope

**In scope:** What a new contributor encounters — README, setup and contributing docs, scripts, config, and error messages on the documented happy path.
**Out of scope:** Code quality, architecture, and test coverage. A confusing implementation is only in scope if a newcomer would hit it while following the documented path.
**Stop when:** All five stages below have been walked, or you have said which you could not complete and what blocked you.

Report findings only — do not edit documentation or fix the setup problems you find.

## Areas to Investigate

### 1. First Impression (0-2 minutes)
- Is the project purpose clear immediately?
- Is the README welcoming and oriented to newcomers?
- Are there quick examples to understand value?

### 2. Setup Process (2-10 minutes)
- Are dependencies clearly listed?
- Do setup commands actually work?
- Are there multiple conflicting instruction sources?
- Are there hidden system requirements?

### 3. Codebase Navigation (10-30 minutes)
- Is the directory structure intuitive?
- Is there a CONTRIBUTING.md or architecture guide?
- Are naming conventions consistent and clear?
- Can you find where to add a simple feature?

### 4. Development Workflow (30+ minutes)
- How to run tests? Is it documented and does it work?
- Hot reload / fast feedback loops?
- How to debug? Any tooling explained?
- Code style enforcement (linters)? Automatic or manual?

### 5. Contribution Process
- How to submit changes? PR template? Guidelines?
- Are there examples of good PRs to learn from?
- Review process transparent?

## Output Format

### Friction Score
**Overall: [Low/Medium/High]**
- Setup: [score]
- Understanding: [score]
- Contributing: [score]

[One paragraph summary of overall experience]

### Critical Blockers 🚫
[Things that STOP a new contributor]
1. **[Blocker name]**
   - What happens: [specific scenario]
   - Evidence: [file:line or missing file]
   - Impact: [why this is critical]
   - Fix: [specific recommendation]

### Confusion Points 🤔
[Things that SLOW DOWN or CONFUSE contributors]
1. **[Confusion source]**
   - Why it's confusing: [explanation]
   - Where it happens: [context/files]
   - Better approach: [suggestion]

### Missing Guidance 📚
[Documentation gaps that would help newcomers]
- [ ] Missing X explanation (needed in file/doc Y)
- [ ] No examples of Z (would help with understanding W)

### Quick Wins 🎯
[Small changes that would significantly reduce friction]
1. **[Quick win]** - [why it helps] - [where to add it]
   - Effort: [Low/Medium]
   - Impact: [High/Medium]


## Section Sizing

Give each part of the report the space it earns:

- **Summary / assessment** — one paragraph, 3-5 sentences.
- **Findings** — one entry per real finding, no cap. Report everything meeting the bar above; do not select a "top N" and drop the rest.
- **Recommendations** — 3-5 items, highest impact first.

Length is not evidence of thoroughness. A finding with a `file:line` and a concrete consequence beats a paragraph of context.

---

**Instructions:**
- Think from a beginner's perspective (don't assume expert knowledge)
- Be specific about WHERE confusion happens (file paths, doc sections)
- Prioritize based on impact to newcomer success
- Quick wins should be actionable (not "improve docs" but "add setup example to README:45")
- If onboarding is smooth, celebrate what works well!