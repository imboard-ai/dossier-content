---dossier
{
  "dossier_schema_version": "1.0.0",
  "title": "Idea to PRD — PM discovery loop with a go/kill gate",
  "version": "0.1.0",
  "protocol_version": "1.0",
  "status": "Draft",
  "last_updated": "2026-07-07",
  "objective": "Take any high-level product idea and EITHER kill it with a documented rationale when ROI/viability logic says it isn't worth chasing, OR generate the rationale for why it makes sense and drive it all the way to an engineering-ready PRD (user benefits + product feature design + text wireframes), tracked in GitHub — not gated in a chat session",
  "category": [
    "development"
  ],
  "tags": [
    "product-management",
    "pm",
    "discovery",
    "prd",
    "go-kill-gate",
    "wireframes",
    "workflow",
    "autonomous",
    "loop",
    "grounded"
  ],
  "risk_level": "medium",
  "requires_approval": false,
  "risk_factors": [
    "modifies_files",
    "network_access",
    "executes_external_code"
  ],
  "destructive_operations": [
    "Creates GitHub issues (epic + optional sub-issues, or a kill record) and may open a docs PR",
    "Writes PRD and wireframe docs into the repo",
    "Spawns background agents that run PM-framework skills autonomously"
  ],
  "inputs": {
    "required": [
      {
        "name": "idea",
        "description": "A high-level product idea, in one or a few sentences. Can be vague; the loop sharpens it.",
        "type": "string",
        "example": "Help CEOs decide when and how to raise their next round"
      }
    ],
    "optional": [
      {
        "name": "slug",
        "description": "kebab-case slug for the feature; used for the docs folder and branch. Derived from the idea if omitted.",
        "type": "string"
      },
      {
        "name": "grounding_docs",
        "description": "Extra strategy-registry docs to load beyond brief.md (e.g. icp.md, gtm-plan.md).",
        "type": "string"
      },
      {
        "name": "autonomy",
        "description": "'checkpoint' (default) = stop for a human sign-off before shipping to GitHub. 'auto' = run straight through to the GitHub issue.",
        "type": "string",
        "default": "checkpoint"
      },
      {
        "name": "kill_threshold",
        "description": "How aggressive the go/kill gate is: 'lenient' | 'balanced' (default) | 'strict'. Strict kills more.",
        "type": "string",
        "default": "balanced"
      },
      {
        "name": "max_critic_iterations",
        "description": "Cap on the PRD critic revise-loop before shipping as-is with the open gaps logged.",
        "type": "number",
        "default": 3
      }
    ]
  },
  "outputs": {
    "files": [
      {
        "path": "docs/features/{slug}/prd.md",
        "description": "The engineering-ready PRD (1-10 skeleton, User Benefits + Product Feature Design)",
        "format": "markdown"
      },
      {
        "path": "docs/features/{slug}/wireframes.md",
        "description": "Text/ASCII wireframes for every screen, house-style",
        "format": "markdown"
      }
    ]
  },
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "name": "idea-to-prd",
  "checksum": {
    "algorithm": "sha256",
    "hash": "c8259b979e69f2778cf297a587e1943065dcc25bd8a518c492b7f305f096b7cc"
  }
}
---

# Idea to PRD — PM discovery loop with a go/kill gate

## Objective

Generalize imboard's PM activity into one repeatable loop: from a **high-level idea** to one of two honest terminal states —

1. **KILL** — a documented decision that the idea is not worth chasing (weak problem, poor ROI, wrong-market, riskiest assumption unlikely to survive), recorded so it is not silently re-litigated; **or**
2. **PRD** — the generated rationale for *why it makes sense*, driven all the way to an **engineering-ready PRD** (explicit user benefits + product feature design + text wireframes of every screen), converged past a quality gate and **shipped to GitHub** so it lives outside any one chat session.

This is a **funnel, not a conveyor belt** — most ideas should not become PRDs. The go/kill gate is the point.

## Guiding Principles

- **Grounded, not generic.** Every stage runs against imboard's real business context (`brief.md`), never generic best-practice.
- **Kill is a first-class outcome.** A well-argued kill is a success. Never quietly drop an idea — record the reasoning and the revisit trigger.
- **Show the work.** Both the go/kill rationale and the PRD are arguments with assumptions and confidence, not verdicts.
- **Un-gate the output.** The durable artifact is a GitHub issue (+ committed doc), not a message in a session.
- **Plan, then push.** Default autonomy stops once for a human sign-off before shipping; otherwise it runs itself.

## Prerequisites

- [ ] The PM-framework skills are installed and enabled (`pm-frameworks-on` + `pm-decisions-on`), and `pm-business-context` can load `brief.md`.
- [ ] `gh` CLI authenticated with push + issue access.
- [ ] The repo has a `docs/features/` convention and a wireframe exemplar to match.
- [ ] Environment can spawn background agents (one per stage) — orchestrate with the Workflow engine where available.

## Phase 0: Ground

Load `pm-business-context` (`brief.md` + any `grounding_docs`). Fix the source of truth: the master brief, not stale in-repo product docs. State the ICP, positioning, stage, and hard constraints the idea must live within. Identify the **live data the feature would reason over** (map to real product entities/KPIs) — an idea that needs data the product does not and will not have is a feasibility flag for the gate.

## Phase 1: Frame

Apply, in order: `problem-clarity` → `problem-statement` (user-centered) → `jobs-to-be-done` (functional + emotional) → `outcome-definition` (metric/baseline/target/timeframe) → `assumption-mapper` (the single highest-risk assumption across desirability/feasibility/viability + how to validate). Synthesize explicit **User Benefits**. Output: is there a real, grounded problem worth a decision?

## Phase 2: Size & Assess

Quantify enough to decide. Use the decision-appropriate subset: `value-vs-effort`, `prioritization-advisor`, `feature-investment-advisor` (ROI), `tam-sam-som-calculator` / `user-segment-prioritizer` (market shape), `saas-economics-efficiency-metrics` (unit economics / runway impact). Produce: value estimate, effort/cost estimate, strategic-fit read against the brief, and the survivability of the riskiest assumption.

## Phase 3: GO / KILL GATE  ← the spine

Decide explicitly, applying `kill_threshold`. Weigh: **problem strength** (Phase 1) · **ROI** (value ÷ effort, Phase 2) · **strategic fit** (does it serve the ICP / positioning / stage, or fight a stated constraint or anti-persona) · **riskiest-assumption survivability** · **opportunity cost**. Emit a decision with reasoning and a confidence level.

- **KILL** → write a short **kill memo**: the idea, the disqualifying logic, what would change the call, and a revisit trigger. File it as a GitHub issue labeled `parked`/`wontfix` (or append to a "considered & killed" log). **STOP.** This is a valid, valuable terminal state — do not soften a kill into a half-hearted PRD.
- **GO** → write the **investment thesis** (the generated "why this makes sense": the problem, the bet, the expected outcome, why now, why us). Continue.

Never let an idea leak past a weak gate. When the threshold is `strict` and the case is marginal, kill and record the trigger to reconsider.

## Phase 4: Strategize (GO only)

Define the solution approach and — for reasoning/advisory features — the **reasoning the feature itself performs** on live data: inputs (mapped to real KPIs/entities), the frameworks it applies, and the **decision-memo output format** (options + assumptions + confidence — an argued memo, never a single oracle answer).

## Phase 5: Design + Wireframe (GO only)

`storyboard` (6-frame user journey) → `user-story-mapping` (screen inventory + MVP-first release slices) → **WIREFRAME every screen as text/ASCII in the repo's house style** (read the wireframe exemplar first; box-drawing chars, per-state variants, `←` behavior annotations, widgets bound to real data) → `user-story` + `user-story-splitting` (Gherkin acceptance). The wireframe stage is bespoke — no PM skill emits screen layouts, so synthesize them from the storyboard + story map.

## Phase 6: Assemble PRD (GO only)

Drive `write-prd` → `prd-development` into the repo's canonical PRD skeleton at `docs/features/{slug}/prd.md` (+ `wireframes.md`). Match the house style. Carry two **named** sections explicitly: **User Benefits** and **Product Feature Design** (embed the full wireframes in the Solution Overview). Call the quant + design skills explicitly — the `write-prd` auto-chain omits them.

## Phase 7: Critic Gate — converge (GO only)

Run `prd-critic` (Ready | Needs Revision) + `launch-readiness`. If not Ready, turn the gaps + rewrite recommendations into revision notes and loop back to the weakest phase (4–6) → re-assemble → re-critique, up to `max_critic_iterations`. "Ready" means an engineer could build it with no clarifying questions, benefits are explicit, and **every screen has a wireframe**. If the cap is hit, ship as-is but **log the residual gaps** in the issue — never present a capped draft as converged.

## Phase 8: Checkpoint (if autonomy = checkpoint)

Surface the converged PRD + critic verdict + top open questions for a human sign-off. Feedback re-enters Phase 7; approval proceeds to ship. This is what keeps the strategy human-led. Under `autonomy = auto`, skip straight to Phase 9.

## Phase 9: Ship to GitHub — un-gate

- Commit `docs/features/{slug}/{prd.md,wireframes.md}` on a branch and open a docs PR (or commit per repo policy).
- Create a GitHub **epic issue**: the investment thesis + user benefits + a link to the PRD doc + the success metrics + the release slices as a checklist (or spawn one sub-issue per MVP slice).
- Optionally hand the sub-issues to `fleet-cycle` / `full-cycle-issue` to build.
- Report the issue URL and PR/doc links. The idea now lives in GitHub, not the session.

## Known Pitfalls

- **Gate erosion.** The failure mode is a gate that always says GO. If nothing ever gets killed, the threshold is broken — calibrate so marginal ideas die.
- **Ungrounded reasoning.** Skipping Phase 0 yields generic PRDs. Always cite the brief; flag when the idea needs data the product won't have.
- **Silent kills.** Dropping an idea without a recorded rationale means it returns forever. Always file the kill memo + revisit trigger.
- **Wireframe skip.** No PM skill produces screen layouts; if you don't synthesize them, "Ready" is a lie. Every screen gets a wireframe.
- **Capped-draft-as-done.** Hitting the critic cap is not convergence — ship with residual gaps logged, loudly.
- **Session-gated output.** A PRD that only exists in the chat is lost. Phase 9 is mandatory for GO.

## Decision Points

| Situation | Decision |
|---|---|
| Weak problem OR poor ROI OR fights a brief constraint/anti-persona | KILL with a memo + revisit trigger |
| Marginal case under `strict` threshold | KILL; record the trigger to reconsider |
| Strong problem + positive ROI + strategic fit | GO; write the investment thesis |
| Critic returns Needs Revision | Loop back to weakest phase (4–6), up to the cap |
| Critic cap hit, still not Ready | Ship with residual gaps logged in the issue |
| `autonomy = checkpoint` | Stop once before Phase 9 for sign-off |

## Validation

- [ ] Grounded in `brief.md` (cited), not generic
- [ ] Explicit GO/KILL decision with reasoning + confidence
- [ ] KILL → kill memo filed with revisit trigger; loop stops
- [ ] GO → investment thesis generated
- [ ] PRD uses the house skeleton with named User Benefits + Product Feature Design sections
- [ ] Text wireframe for every screen in the story map
- [ ] Critic = Ready (or capped with gaps logged)
- [ ] Output shipped to a GitHub issue (+ doc), not left in-session

## Relationship to Other Dossiers

- **Composes**: the PM-framework skills (`pmf-*` / `pmd-*`) as its per-phase reasoning stages.
- **Feeds**: `fleet-cycle` / `full-cycle-issue` — the GO path's decomposed sub-issues are built by the issue-workflow family.
- **Sits beside**: `feature-to-issues` (decomposition) — this dossier is the discovery-and-decision layer that produces the epic those decompose.
