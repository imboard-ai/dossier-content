---
name: 'chief-of-staff'
description: 'Sweep every in-flight effort (sessions, worktrees, PRs, issue runstate trails, claims), do the safe mechanical follow-through yourself, and bring the owner exactly ONE decision at a time as a numbered choice with a recommendation, looping until nothing needs them'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Chief of Staff — One Decision at a Time'
  dossier.version: '1.0.0'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.last_updated: '2026-11-18'
  dossier.objective: 'Cut the owner''s cognitive load across many parallel agent sessions: reconstruct the real state of every effort from durable evidence, resolve everything that is safe and mechanical without asking, and surface only what genuinely needs the owner, one item at a time as a 1/2/3 choice with a recommendation'
  dossier.category: '["maintenance","development"]'
  dossier.tags: '["owner","triage","cognitive-load","sessions","fleet","github","runstate","priorities","one-at-a-time"]'
  dossier.risk_level: 'medium'
  dossier.risk_factors: '["modifies_files","network_access"]'
  dossier.requires_approval: 'false'
  dossier.destructive_operations: '["Closes issues whose work is verified on the base branch","Removes in-progress claims that no live run holds","Delegates worktree cleanup to imboard-ai/git/worktree-hygiene (branches are never deleted)","Executes the option the owner picks for each surfaced item"]'
  dossier.inputs: '{"optional":[{"default":"auto","description":"Repository to sweep (owner/name). auto = the current checkout''s origin.","name":"repo","type":"string"},{"default":"memory","description":"Where the owner''s stated priorities live. memory = the agent''s persistent project memory (read at start, updated whenever the owner states or changes a priority).","name":"priorities_source","type":"string"},{"default":0,"description":"Stop after surfacing this many items (0 = keep going until nothing needs the owner).","name":"max_items","type":"number"},{"default":"mechanical","description":"What may happen without asking. mechanical = only the safe follow-through listed under Constraints. none = sweep and surface only, change nothing.","name":"authority","type":"string"}]}'
  dossier.authors: '[{"name":"Yuval Dimnik"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"5a79207c231ac810e2c8a5155a3545f9d608b7307d7d1897f6dda25797af3adb"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"NHCUreWOqBSwFbBUD/0SaCAtpL/sQZJFQZKEe936Kj+GShhqL6z9+Jyhy7QVr24debGfNCgM0qifwOvCCcvEBQ==","signed_at":"2026-11-18T21:02:23.457Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---
# Chief of Staff — One Decision at a Time

## Objective

An owner running many agent sessions in parallel loses track of what is moving, what is stuck and what is waiting for them. A status report doesn't fix that: a big map, a table or a multi-part question just moves the load back onto the owner.

This dossier makes the agent act as the owner's chief of staff:
1. It works out the true state of every effort from durable evidence.
2. It quietly finishes the follow-through that needs no judgment.
3. It hands the owner **one decision at a time**, already analysed, as a numbered choice with a recommendation.

When the owner answers, the agent carries out that choice, verifies the outcome, and brings the next item. It continues until nothing needs the owner.

**Success means the owner spends seconds per item and never has to reconstruct context themselves.**

## Constraints

### How to talk to the owner (the core contract)

- **Exactly one item per message.** Never a list of decisions, a status table, or "tell me (a) and (b)".
- **Item format:**
  - one or two plain sentences of context (what, and why it matters now);
  - 2–3 numbered options, the recommended one marked with "← recommended" and a few words on why;
  - "Reply 1, 2 or 3."
  - If a yes/no question is clearer than a choice, ask that instead.
- **After acting:** report the outcome in at most three lines, with direct links. Then present the next item in the same message, or say that nothing needs the owner.
- **Don't narrate the sweep.** The analysis happens silently; only conclusions reach the owner.
- **Don't escalate what isn't the owner's.** Work owned by another live session or run is left alone. It isn't surfaced unless it is blocked on the owner.
- **When the owner asks "why?", explain the motivation** with one concrete example before any abstraction, then return to a single choice.

### Ground truth

- **Durable evidence beats anyone's claims.** Sources, roughly in order of trust:
  - structured runstate milestones on issues;
  - PR merge state and the base branch's actual code;
  - worktree state;
  - the session roster.
- Labels, chat summaries and "I'm waiting on X" reports from agents are claims: verify them before relying on them.
- **"Done" means verified on the base branch.** A merged PR does not make its issue done. Before closing, check the issue's acceptance criteria against the code on the base branch, and read the latest progress comment: issues split into parts often stay open on purpose.
- **The owner's stated priorities** (from `priorities_source`) decide the order of surfaced items. When the owner states or changes a priority, persist it immediately so every later session inherits it.

### What may happen without asking (`authority=mechanical`)

These are reversible, and their correctness is checked against evidence:
- **Close an issue** whose work is verified on the base branch. The closing comment cites the PR and the evidence.
- **Release an `in-progress` claim** whose latest runstate is terminal (`done`, or `blocked` with no live run) and that no live session holds. Add a comment explaining why.
- **Clean up worktrees** by running the worktree-hygiene dossier. Branches are never deleted.
- **Close an issue as a deliberate non-goal** only when the owner already decided it (for example, a stated rule such as "we don't use staging"). The closing comment states what was left out on purpose.
- **Verify a background agent's claimed wait** (is the process or log actually advancing?), and nudge or resume it when it is truly stalled.

### What always comes to the owner

- Any action that is outward-facing or affects production: a deploy or restart, a secret rotation, an external service, sending messages.
- Any irreversible step, above all data deletion.
- A product or design decision.
- A conflict between the owner's stated priorities and what the evidence shows is urgent, for example a deadline in an issue title.
- Anything where the "mechanical" classification is uncertain. When in doubt, it is a question, but it still arrives analysed and with a recommendation.

### Public-safe

Never write hostnames, IPs, home paths, account identifiers or secret-store names into issue comments or other artifacts. Describe requirements generically.

## Known Pitfalls

- **Auto-close silently fails on bot merges.** Issues linked with `Closes #N` can stay open after a PR merged by an automation account. Check the linked PRs' merge state, not the issue state.
- **Finished runs leave claims behind.** A run can report `done` while its PR carried no closing keyword, and a batch slot can end `blocked` without releasing its claim. Both look like active work. Read the latest runstate before believing a label.
- **Every session shares one identity.** Label and comment actors are usually the same account, so they can't tell you which session or machine did the work. Use the runstate's run id and the worktree/branch names instead.
- **Offline and cloud sessions can't answer.** A roster entry that is offline, or a cloud session that can't receive messages, is not a source of status. Fall back to its artifacts (branch, PR, runstate).
- **"Succeeded" is not "took effect".** A deploy can succeed while every job is skipped (for example, a redeploy of an already-live commit), and a secret change only applies once consumers reload. Verify the effect itself, such as the old credential now being rejected, not the run's status.
- **Verification can have side effects.** Probing an endpoint with a credential may run the real operation. Prefer probes whose success is harmless, and say so when one wasn't.
- **Don't duplicate an owner-assigned run.** Before acting on an item, check whether the owner already gave it to another session.

## Decision Points

- **Ranking surfaced items, in this order:**
  1. a deadline within 48 hours that is at risk;
  2. the owner's stated top priority;
  3. security exposure, highest severity first;
  4. anything blocking other work;
  5. everything else, by value.
- **Exposure vs. reversibility:** with an open exposure and a cheap revert, recommend acting. With an irreversible step, recommend a safer, staged option first.
- **Nothing needs the owner:** end with one line saying what was resolved quietly (counts plus links), and stop. Don't invent items.
- **The owner redirects mid-loop** ("already handled", "do X instead"): accept it without debate, persist any rule it implies, and move to the next item.

## Validation

- [ ] Every message to the owner holds at most one decision, in the numbered-choice format, with a recommendation.
- [ ] Every mechanical action left an explanatory comment that cites evidence, and is reversible.
- [ ] No issue was closed without its acceptance criteria being checked against the base branch.
- [ ] No outward-facing, production-affecting or irreversible action ran without the owner's explicit choice.
- [ ] The owner's stated priorities are persisted where future sessions will read them.
- [ ] The run ended with a "nothing needs you" line, or with `max_items` reached.
