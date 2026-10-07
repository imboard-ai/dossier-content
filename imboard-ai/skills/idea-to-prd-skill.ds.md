---
name: 'idea-to-prd-skill'
description: 'Run the PM discovery loop with a go/kill gate. Frames an idea, sizes it, then either KILLS it with a documented rationale (weak problem / poor ROI / fights the brief) or generates the rationale and drives it to an engineering-ready PRD with text wireframes, then ships it to a GitHub issue. Use when the user says ''idea to prd'', ''run the idea-to-prd loop'', ''spec this idea'', ''should we build X'', ''is this worth building'', ''kill or prd'', ''take this idea to a PRD'', ''strategy to prd'', or hands over a high-level product/feature/strategy idea to evaluate and (maybe) specify.'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Idea to PRD'
  dossier.version: '0.1.0'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.risk_level: 'medium'
  dossier.requires_approval: 'false'
  dossier.objective: 'Take a high-level product idea to a documented KILL or an engineering-ready PRD (benefits + feature design + text wireframes), grounded in the business brief and shipped to GitHub'
  dossier.category: '["skills"]'
  dossier.tags: '["product-management","pm","discovery","prd","go-kill-gate","wireframes","workflow","skill"]'
  dossier.authors: '[{"name":"Yuval Dimnik"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"d63d15253ef1ac4db759425ea6e3e0486ff45791b1e39fa96f17d1def54f62c7"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"ZVpQXAvCmnzCKebpaz0/2u+2+n/R4QjZm7SV7q0sJghlw2J6/qqOJy90EFLmnCWlSJLK36OI8ISKxs99T6tEAw==","signed_at":"2026-10-07T14:09:11.686Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---

# Idea to PRD

Take any high-level idea to one of two honest ends — a documented **KILL**, or an engineering-ready **PRD** (user benefits + product feature design + text wireframes) shipped to GitHub. This is the PM discovery layer; the go/kill gate is the point (a funnel, not a conveyor belt).

## Project Parameters

- **grounding source of truth**: the business brief loaded by the `pm-business-context` skill (`brief.md`) — NOT the stale in-repo `product.md`.
- **PRD home + house style**: `main/docs/features/<slug>/prd.md` + `wireframes.md`, canonical 1–10 skeleton (exemplar `investor-intelligence-hook.md`); wireframe style exemplar `ai-powered-company-onboarding/wireframes.md`.
- **PM skill packs**: ensure both `pm-frameworks-on` and `pm-decisions-on` are enabled.

## Inputs

Parse from the user's request:
- `idea` (required): the high-level idea, one or a few sentences.
- `slug` (optional): kebab-case; derived from the idea if omitted.
- `grounding_docs` (optional): extra strategy-registry docs beyond brief.md (e.g. icp.md, gtm-plan.md).
- `autonomy` (default `checkpoint`): stop once for human sign-off before shipping to GitHub; `auto` runs straight through.
- `kill_threshold` (default `balanced`): `lenient | balanced | strict`.

## Steps

1. Run: `ai-dossier run imboard-ai/pm/idea-to-prd --pull` and pass through `idea` + the optional inputs above.
2. Follow ALL phases in the dossier output, in order: Ground → Frame → Size & Assess → **GO/KILL GATE** → (KILL: file kill memo + revisit trigger, STOP) → (GO: investment thesis) → Strategize → Design + Wireframe → Assemble PRD → Critic Gate (loop-until-Ready, capped) → Checkpoint → Ship to GitHub. Do not skip the gate or the wireframe stage.
3. **Orchestrate the per-decision stages** as background agents (or a Workflow pipeline): each stage runs its mapped PM-framework skill grounded in `brief.md`; the critic gate loops back to the weakest stage until `prd-critic = Ready` or the cap, logging residual gaps if capped.
4. **Kill is a first-class outcome.** If the gate says KILL, produce the kill memo (disqualifying logic + what would change the call + revisit trigger), file it as a `parked` GitHub issue, and STOP — do not soften into a half-hearted PRD.
5. **Un-gate the output.** For GO, commit `docs/features/<slug>/{prd.md,wireframes.md}` on a branch (docs PR) and create a GitHub **epic issue** with the thesis, benefits, PRD link, metrics, release-slice checklist, and — if the critic capped at Needs-Revision — the residual gaps as a pre-build checklist. Report the issue + PR URLs.
6. **Checkpoint** (default autonomy): before shipping, surface the converged PRD + critic verdict + top open questions for a human sign-off; approval proceeds to ship. Under `autonomy = auto`, skip straight to ship.

## Notes

- Concrete reference implementation: the reusable Workflow at `strategy-to-prd.workflow.js` (parameterized by decision via `args`) implements Frame → Strategize → Design+Wireframe → Assemble → Critic-gate; first two runs = Fundraise-Timing Advisor (#2614) and Growth-vs-Cash Advisor.
