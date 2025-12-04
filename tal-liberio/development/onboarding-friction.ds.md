---
authors:
- name: tal
category:
- documentation
- development
checksum:
  algorithm: sha256
  hash: 9b173f04c783b8ca9141c595a9ad393226fc254caf2a45ac783ac89212b58cf4
estimated_duration:
  max_minutes: 5
  min_minutes: 2
name: onboarding-friction
objective: Identify pain points and confusion for new contributors trying to understand
  and work with the project
schema_version: 1.0.0
signature:
  algorithm: ed25519
  public_key: aFl367Ta9sFHpQZXSsndkGYm4h4BgSuCV/f/qkMIvl4=
  signature: VUP7YaH6O4VoLMhxqwdnhO2e58tbAGMIIBiR2iG0dED2jPDcP0Zf/fp7QdTMvFZQg53baFcOx0qaGl064spRAQ==
  signed_by: tal@libero.ai
  timestamp: '2025-12-04T16:43:32.621920+00:00'
status: draft
tags:
- onboarding
- contributor-experience
- analysis
title: Onboarding Friction Assessment
version: 1.0.0
---

---
authors:
- name: tal
category:
- documentation
- development
checksum:
  algorithm: sha256
  hash: c9c593fb870999442a51dd8f18a8d65d3b07eaa5058a276f5e95aa0610cf68c4
estimated_duration:
  max_minutes: 5
  min_minutes: 2
objective: Identify pain points and confusion for new contributors trying to understand
  and work with the project
schema_version: 1.0.0
signature:
  algorithm: ed25519
  public_key: aFl367Ta9sFHpQZXSsndkGYm4h4BgSuCV/f/qkMIvl4=
  signature: 9QOzLsSC/cy4+lLYszDcxZfzZMz42eEhXP/zW77BwrzcmVLwgT3HXEZlNxBnWogSpsIb6hdJ/6NB7IDPOywxAQ==
  signed_by: tal@libero.ai
  timestamp: '2025-12-04T16:26:09.591646+00:00'
status: draft
tags:
- onboarding
- contributor-experience
- analysis
title: Onboarding Friction Assessment
version: 1.0.0
---

---
authors:
- tal
category:
- documentation
- development
checksum:
  algorithm: sha256
  hash: 19206b141359b53b6f2d85b545d07a2f7dac5e3f4a2a3611e3f65b35bb77a858
estimated_duration:
  max_minutes: 5
  min_minutes: 2
objective: Identify pain points and confusion for new contributors trying to understand
  and work with the project
schema_version: 1.0.0
signature:
  algorithm: ed25519
  public_key: aFl367Ta9sFHpQZXSsndkGYm4h4BgSuCV/f/qkMIvl4=
  signature: 4yivHKSaQc2Bt9R5PEau7FEpq2OOzWt2ZDHZz39QWR8KbI8oNftNARKPj42qC7byk/DQTMHbbIUXF7Mv5LheAQ==
  signed_by: tal@libero.ai
  timestamp: '2025-12-04T16:22:59.901198+00:00'
status: draft
tags:
- onboarding
- contributor-experience
- analysis
title: Onboarding Friction Assessment
version: 1.0.0
---

---
authors:
- tal
category:
- documentation
- development
checksum:
  algorithm: sha256
  hash: ccc869a98e11f428607528d530706d2fd2cf325c72a37635a5339eb54ecf3755
estimated_duration:
  max_minutes: 5
  min_minutes: 2
objective: Identify pain points and confusion for new contributors trying to understand
  and work with the project
schema_version: 1.0.0
status: draft
tags:
- onboarding
- contributor-experience
- analysis
title: Onboarding Friction Assessment
version: 1.0.0
---

# Onboarding Friction Assessment

Evaluate the new contributor experience by simulating a newcomer's journey through the project.

## Your Task

Simulate the journey of a new contributor who wants to:
1. Understand what this project does
2. Set up a development environment
3. Understand the codebase structure
4. Make their first contribution
5. Run tests and verify their changes

For each stage, identify friction points.

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

### Critical Blockers
[Things that STOP a new contributor]

1. **[Blocker name]**
   - What happens: [specific scenario]
   - Evidence: [file:line or missing file]
   - Impact: [why this is critical]
   - Fix: [specific recommendation]

### Confusion Points
[Things that SLOW DOWN or CONFUSE contributors]

1. **[Confusion source]**
   - Why it's confusing: [explanation]
   - Where it happens: [context/files]
   - Better approach: [suggestion]

### Missing Guidance
[Documentation gaps that would help newcomers]

- [ ] Missing X explanation (needed in file/doc Y)
- [ ] No examples of Z (would help with understanding W)

### Quick Wins
[Small changes with high impact on contributor experience]

1. **[Quick win]** - [why it helps] - [where to add it]
   - Effort: [Low/Medium]
   - Impact: [High/Medium]

## Notes

- Think from a beginner's perspective—don't assume expert knowledge
- Be specific about WHERE confusion happens (file paths, doc sections)
- Prioritize based on impact to newcomer success
- Quick wins should be actionable ("add setup example to README:45" not "improve docs")
- If onboarding is smooth, celebrate what works well