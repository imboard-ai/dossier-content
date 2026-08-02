---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "sheet-triage",
  "title": "QA Sheet Triage",
  "version": "1.2.0",
  "protocol_version": "1.0",
  "status": "Draft",
  "last_updated": "2026-08-01",
  "objective": "Turn a manual-QA findings spreadsheet into verified, clustered GitHub issues plus the detection-gap fixes that would have caught each bug automatically",
  "description": "Ingest a manual-QA findings sheet, validate that every referenced piece of evidence is actually readable, surface-review the findings for discrepancies, cluster them by the code surface a fix would touch, investigate each cluster with parallel root-cause / generalization / detection-gap agents, file one issue per cluster, and write the triage result back as a companion sheet. Use when the user says 'triage QA sheet', 'QA findings', 'manual QA results', or hands over a spreadsheet of bugs.",
  "category": [
    "testing",
    "maintenance"
  ],
  "tags": [
    "qa",
    "triage",
    "github",
    "workflow",
    "testing",
    "root-cause"
  ],
  "risk_level": "medium",
  "requires_approval": false,
  "risk_factors": [
    "network_access",
    "requires_credentials"
  ],
  "destructive_operations": [
    "Creates GitHub issues in the target repository (team-visible, noisy to undo)",
    "Creates a new companion spreadsheet in the QA evidence folder"
  ],
  "estimated_duration": {
    "min_minutes": 20,
    "max_minutes": 120
  },
  "tools_required": [
    {
      "name": "gh",
      "description": "GitHub CLI, authenticated against the target repository",
      "check_command": "gh auth status"
    }
  ],
  "inputs": {
    "required": [
      {
        "name": "sheet",
        "description": "The QA findings source: a spreadsheet URL or ID reachable through the agent's document connector, a local .csv/.tsv path, or pasted tabular text",
        "type": "string",
        "example": "https://docs.google.com/spreadsheets/d/1Gz8.../edit"
      }
    ],
    "optional": [
      {
        "name": "repo",
        "description": "Target GitHub repository in owner/name form. Defaults to the current repository's origin remote.",
        "type": "string",
        "default": "auto"
      },
      {
        "name": "dry_run",
        "description": "Run every investigation phase and print the full report, but create no issues and no companion sheet",
        "type": "boolean",
        "default": false
      },
      {
        "name": "only",
        "description": "Comma-separated finding IDs to process, ignoring the rest of the sheet",
        "type": "string",
        "default": ""
      },
      {
        "name": "no_writeback",
        "description": "Skip Phase 5 — file issues but create no companion sheet",
        "type": "boolean",
        "default": false
      },
      {
        "name": "checkpoint",
        "description": "Always stop for review after the Phase 1 surface review, even when nothing is ambiguous",
        "type": "boolean",
        "default": false
      }
    ]
  },
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "948472a21722ff6795ea45dc5d3c6ab00c6c5088ebfef6cde68bc1e4e1754ec0"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "WHr0RkhdkSWZKGKyn+EvSGbp8fpcGls6EIdqPvOp9jhWmkbil6BvdYO31BoyhKIxF+5oHGqNfW9J7IgPFEwZAw==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-08-02T10:16:57.970Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# QA Sheet Triage

## Objective

A human tester hands over a sheet of findings. **The findings are not the deliverable —
the misses are.**

Every finding on that sheet is a defect that reached a human because no automated check
caught it first. Fixing the bug closes one instance. Naming and closing the gap that let
it through retires the class. Produce both, but understand which one you are actually
here for.

This matters most where the product has few or no users: automated checking is the *only*
thing standing between a defect and production, so a human finding a bug is already a
system failure, independent of how bad the bug is. Treat a four-month-old 100%-failing
workflow and a cosmetic typo as the same category of question — *why did nothing catch
this?*

Measure a round by **classes of defect retired**, not by bugs found.

## Guiding Principle

**Measured, not reasoned.** Every claim you make about the code is either CONFIRMED —
you read the file, ran the command, saw the output — or SUSPECTED. Label each one. A
tester's description is a report of symptoms, never a diagnosis.

**A proposed artifact is not a closed gap.** An issue that says "we should add a test" has
retired nothing. The gap closes when the artifact exists and runs — verify that, and carry
anything unverified into the next round rather than counting it as done.

**Do not stop to ask mechanical questions.** Issue titles, cluster names, label choices,
file paths and ordering are yours to decide. Pause only where this dossier says to.

**Stop at filed issues.** This workflow does not change product code. Fixing is a
separate, deliberately-triggered workflow.

## Prerequisites

- GitHub CLI (`gh`) installed and authenticated against the target repository.
- Read access to the findings sheet and to every file it references.
- You are inside the target repository's working tree.

## Actions to Perform

### Phase 0: Access & Integrity Gate

Prove you can actually read everything before you reason about any of it. A silent
partial read is the failure mode this phase exists to prevent.

1. Resolve the sheet from the `sheet` input. If none was given, search the document
   store by likely title and report candidates rather than guessing.
2. Fetch its metadata: confirm it is a spreadsheet, and record owner, parent folder and
   last-modified time.
3. Read the full grid, including any legend, instructions or "how to use" tab. Those
   tabs usually carry the evidence-folder link and the severity/status vocabularies.
4. **Resolve every referenced artifact.** Screenshots, recordings and linked documents
   are the evidence for the findings. For each: fetch metadata, and read the content of
   every image (or a representative sample when there are more than 15). You are
   checking two different things — that the artifact is *readable*, and that it *shows
   what the row claims it shows*.
5. List the evidence folder itself and compare its contents against the links in the
   sheet. Per-file sharing frequently means the folder listing is incomplete; say so
   rather than assuming the missing files do not exist.
6. Search for duplicate copies of the sheet. Testers often work in their own copy while
   the recipient holds a separate one. State plainly which copy you read and which
   folder any write-back will target.

**Hard block**: if the sheet itself cannot be read, stop and report. Do not proceed on a
partial read.

Otherwise emit a per-artifact status table — readable / empty / unreadable / missing —
and carry it forward. Unreadable evidence is a finding about the QA process and belongs
in the final report.

### Phase 1: Surface Review

A fast pass over the findings as *written*. No code reading yet.

Normalize the rows into a table, then flag:

- Repro steps that are incomplete, or that do not actually reach the described state.
- Missing evidence, or evidence that contradicts the written claim.
- Severity or priority that disagrees with the described impact.
- **Rows that are not defects** — feature requests, product questions, and copy
  preferences filed as bugs. These are still valuable; they are just not bug issues.
- Duplicates within the sheet.
- **Candidate clusters** — rows sharing a symptom, an error string, or a screen.
- **Already filed or already fixed** — search the issue tracker (open *and* closed) for
  each finding's distinguishing phrase before assuming it is new.

Output the review table plus an explicit **"Needs a decision"** list.

**Pause rule.** Continue automatically for everything unambiguous. Stop and ask only
when the sheet or its evidence is unreadable, a finding is a genuine product decision,
or a finding looks like a duplicate of existing tracker work. When `checkpoint` is set,
always stop here. Every other flag travels forward into the issue body as an
`## Open questions` section — flagging is not the same as blocking.

### Phase 2: Cluster by Code Surface

Group findings by **the code a fix would touch**, not by how they look to a user.

- Same file or module, same fix → **one** issue covering several finding IDs.
- Different surfaces → **separate** issues, cross-referenced.
- Every cluster must be justified with the **shared file path(s)** that make it a
  cluster. A cluster you cannot justify with a path is a guess — split it.
- Default to separate issues when unsure. Over-merging hides work; over-splitting only
  costs a little noise.

Two findings with identical symptoms in different components are usually two issues that
share one detection gap. Two findings with different symptoms emitted from the same
function are usually one issue.

### Phase 3: Investigate — Three Lenses, Each At Its Own Scope

Three lenses, run in parallel. **Each lens has a different natural scope, and getting this
wrong is the most expensive mistake available here:**

| Lens | Scope | Why |
|---|---|---|
| **A — Root cause** | one per cluster | Diagnosis is inherently local to the code a fix touches. |
| **B — Generalization** | one per *pattern* | A pattern spans clusters. Two clusters sharing a defect shape need one search, not two. |
| **C — Detection gap** | **exactly one, over ALL findings** | This is the primary lens, not the third one. Its most valuable output is the gap explaining several findings at once — sharded per cluster it is structurally incapable of seeing that. |

Lens C is the reason the round exists. If effort has to be cut, cut A and B coverage
before cutting C's depth.

Do **not** run a detection-gap agent per cluster. It will report the same shallow
"add a test here" N times and will miss the repeated gap — which is the single
highest-value thing this workflow can produce. One agent, all findings, once.

Identify the patterns for lens B *after* clustering: look for clusters whose root causes
rhyme (same missing guard, same primitive, same contract violation). One B agent per
distinct pattern. If no clusters rhyme, one B agent per cluster is the degenerate case.

**Proportionality.** Match investigation depth to what is at stake. Before launching,
decide and state which mode you are in:
- **Full** — anything user-blocking, data-touching, security-adjacent, or affecting a
  primary flow. All three lenses.
- **Light** — cosmetic, copy, and single-call-site presentation findings. Lens A only,
  plus a single shared lens C for the whole batch. Do not spawn a generalization agent to
  ask whether a typo generalizes.

A round of eleven CSS nits does not deserve the same fleet as a round containing a 500 on
a primary action. Say which mode you chose and why.

---

#### Classification Criteria (applies to ALL investigation agents)

Label every claim:

- **CONFIRMED** — you read the specific file, ran the specific command, and observed the
  result. Cite `file:line` or the command output.
- **SUSPECTED** — consistent with the evidence but not verified. Say what would confirm it.

Never present a SUSPECTED claim in CONFIRMED language. An unlabelled claim is a defect in
your output.

---

#### Agent A: Root Cause

> You are diagnosing a confirmed QA finding. You will be given the finding text, the
> tester's repro steps, and the observed symptom including any verbatim error strings.
>
> Establish what is actually broken:
> 1. Follow the repro path through the code from the entry point the tester used.
> 2. Locate the exact origin of the observed behaviour — for an error string, find where
>    that string is produced, not merely where it is displayed.
> 3. State the **proximate** cause (what fails) separately from the **design** cause
>    (why the code was shaped so this could happen). They are rarely the same, and the
>    second is the one worth fixing.
> 4. Describe the minimal correct fix. Do not implement it.
>
> Cite `file:line` for every structural claim, and classify per the Classification
> Criteria above. If the finding does not reproduce in the code as described, say so
> explicitly — a finding that cannot be traced is a result, not a failure.

#### Agent B: Generalization

> You are checking whether a confirmed defect is an instance of a pattern.
>
> Given the root-cause location, search the **whole** codebase for other sites with the
> same shape — the same missing guard, the same unsafe assumption, the same
> copy-pasted construct. Search by structure, not by the specific identifiers involved;
> the same bug rarely reuses the same variable names.
>
> Report every match as `file:line` with a one-line note on whether it is genuinely the
> same defect or merely similar-looking. Then recommend exactly one of:
> - **Fix centrally now** — a single change resolves all sites,
> - **Fix here, sweep separately** — with the sweep scoped and sized,
> - **Not generalizable** — this is genuinely a one-off, and say why.
>
> Where a lint rule, type constraint, or static guard could make the whole class
> unrepresentable, propose it concretely enough to implement.
>
> Classify per the Classification Criteria above. If nothing else matches, report
> "No other instances found" — that is a real and useful result.

#### Agent C: Detection Gap

> You are answering the question a good bug-fix review asks: **what automated check
> should have caught this before a human did, and why did it not?**
>
> First, learn how this project actually tests itself. Read its contributor guide, its
> QA and testing documentation, its pull-request template, and its CI workflow
> definitions. Understand the layers that already exist before proposing anything — your
> job is to fill a hole in *their* ladder, not to install a new one.
>
> Then determine:
> 1. Which existing layer *should* own this class of defect.
> 2. Why it did not catch this instance. Distinguish sharply between: **no test existed**,
>    **a test existed but did not run** (gating, discovery, or scoping excluded it), and
>    **a test ran but asserted too loosely**. These have completely different fixes.
> 3. The specific artifact that closes the gap.
>
> Your output must terminate in a **named, addable artifact** — a concrete test file
> path, a fixture or scenario registration, a coverage-baseline entry, a lint rule, or a
> CI configuration change. "Add more integration tests" is not an answer. Neither is
> "an end-to-end test would catch it", which is nearly always true and nearly always the
> most expensive way to be right — reach for the cheapest layer that could have caught it.
>
> Note especially any defect that is structurally invisible to the existing layers —
> for example behaviour that only exists while an overlay is open, an element is
> hovered, or data is present. Those gaps recur, and naming the structural reason is
> worth more than naming the single missing test.
>
> Because you see every finding at once, your most valuable output is any gap that
> explains **more than one** of them. Look for it deliberately. Also check whether the
> layer you are about to recommend actually *runs* — a suite that is never invoked, or
> that passes vacuously when its inputs are missing, is a gap and not a safeguard.
>
> Classify per the Classification Criteria above.

---

#### Reporting

Every agent must **send its findings back**. An agent that finishes without reporting has
done no work. If an agent goes idle without delivering, ask it again and require a
partial answer with the unreached parts named — partial findings, clearly labelled, are
useful; silence is not.

---

#### Reconcile Before Believing

Agents disagree, and when they do **they are often both wrong**. Before any finding
reaches an issue:

1. **Diff the overlapping claims.** Where two agents counted, named, or located the same
   thing differently, treat *both* numbers as unverified.
2. **Measure it yourself.** Run the grep, open the file, count the call sites. Do not
   average the estimates, do not pick the more confident agent, and do not quote a
   number in an issue that no one measured.
3. **Prefer the specific over the plausible.** A claim citing `file:line` outranks a
   claim citing a rationale, regardless of which agent sounds more certain.
4. **Carry disagreements forward when they cannot be resolved.** An issue that says "two
   analyses disagreed on scope; verify before sweeping" is honest and useful. An issue
   that silently picks one is a fabrication with a citation.

A number that lands in a GitHub issue is a number someone will act on. Earn it.

#### After All Agents Complete

Collect the three outputs per cluster. Where Agent A could not trace a finding, mark the
cluster as unconfirmed and carry it to the report rather than filing a speculative issue.

Where several clusters share one detection gap, note it — a gap that appears more than
once in a single QA round is the highest-value thing this workflow can find, and it
should be filed once, not repeated in every issue.

### Phase 4: File Issues

Learn the repository's issue conventions before writing anything: read its contributor
guide for which template applies to bug reports, look at two or three recent, well-formed
bug issues to calibrate title and body style, and list the available labels rather than
inventing them.

**One issue per cluster.** Structure the body so a reader gets, in order: what is broken
and how it was found, where it lives, why it happens, what to change, what else is
affected, and how it should have been caught. Concretely:

- **Provenance first** — that this came from manual QA, by whom, when, and a link to the
  sheet. Readers must be able to trace a claim back to its evidence.
- **The tester's evidence verbatim** — exact error strings in backticks, screenshot
  links, the finding IDs it covers. Do not paraphrase an error message.
- **`file:line` for everything structural**, from Agent A.
- **Root cause**, with CONFIRMED / SUSPECTED preserved. Do not launder a hypothesis into
  a fact on the way into the issue.
- **Also affected**, from Agent B — omit the section entirely when nothing matched.
- **Detection gap**, from Agent C — what would have caught it, and why it did not.
- **Open questions**, carrying any unresolved Phase 1 flags — omit when empty.
- **Explicitly out of scope**, so the fix does not sprawl.
- **Acceptance criteria** as checkboxes, including the named detection-gap artifact and
  whatever hygiene/lint gate the project requires before merge.

**Spin off a separate issue** only when Agent B's generalization is substantially larger
than the bug fix, or Agent C's prevention work is its own infrastructure change. Label
those as testing or infrastructure work and cross-reference the bug issue. Do not bury
a codebase-wide sweep inside a single-screen bug fix.

Before creating anything, search the tracker once more for each cluster's key phrase and
skip anything already filed. Then create the issues, and record the resulting numbers
and URLs against their finding IDs.

Never apply labels that belong to pull requests rather than issues — merge-automation,
hold, and CI-trigger labels in particular.

### Phase 5: Write Back a Companion Sheet

The tester needs to see what happened to each row, in the format they work in.

If your document connector cannot edit the original sheet in place — most cannot — create
a **new** companion sheet in the shared evidence folder so the tester can see it, named
so it clearly pairs with the source round. One row per original finding, carrying:

`Finding ID | Cluster | Triage Status | Issue | Issue URL | Root cause (1 line) | Detection gap (1 line) | Notes`

`Triage Status` is one of: Filed / Duplicate of #N / Already fixed in #N / Needs product
decision / Not a bug / Evidence missing / Could not reproduce.

**Every original row appears**, including rejected, duplicate and unconfirmed ones. A row
that silently vanishes reads as an oversight and erodes trust in the whole report.

If creation fails, emit the same table as paste-ready delimited text and say plainly that
the write-back did not happen.

Skip this phase entirely when `no_writeback` or `dry_run` is set.

### Phase 6: Report

Report, in this order:

1. **Coverage** — findings in, clusters out, issues filed, with links.
2. **Prevention** — the detection-gap issues, called out separately from the bug issues.
   This is the compounding half of the work and it should not be buried in a list.
3. **Repeated gaps** — any detection gap that appeared in more than one cluster.
4. **Questions for the tester** — unreadable evidence, missing screenshots, sharing
   problems, findings that could not be reproduced. Include a short, courteous message
   they can act on directly.
5. **Prior-round audit — do this before claiming progress.** Read the cycle log and, for
   every detection-gap artifact promised in previous rounds, check whether it **actually
   exists and runs today**. A named test file that was never written, or written and never
   invoked, is an open gap wearing a closed issue's clothes. Report each as landed /
   proposed-only / landed-but-not-running, and carry the unlanded ones forward.

   This step is what separates a workflow that retires defect classes from one that
   generates confident issue text.

6. **Cycle record** — append this round: date, sheet identifier, findings count, issues
   filed, **artifacts promised**, and **artifacts verified landed**. Those last two are
   different numbers and the gap between them is the honest measure of the process.

## Output

- `findings_total`: number of rows read from the sheet
- `clusters`: list of clusters, each with its finding IDs and justifying file path(s)
- `issues_filed`: list of `{issue_number, url, finding_ids}`
- `prevention_issues`: issues filed for detection-gap or generalization work
- `unconfirmed`: findings that could not be traced to code
- `writeback_url`: link to the companion sheet, or the reason there is none
- `questions_for_tester`: unresolved evidence and reproduction problems

## Known Pitfalls

- **A partial read looks exactly like a clean read.** Evidence links resolve
  individually while the folder listing returns almost nothing; a linked document opens
  but returns empty content. Phase 0 exists because these failures are silent.
- **Row count is not issue count.** Findings cluster, and a well-triaged round usually
  produces meaningfully fewer issues than rows. An issue count equal to the row count
  means Phase 2 did not happen.
- **Symptom-clustering is the tempting mistake.** Two visually identical layout bugs in
  different components are two fixes. Two unrelated-looking errors emitted by the same
  function are one fix.
- **"An end-to-end test would have caught it"** is almost always true, almost always the
  most expensive answer, and usually a sign the analysis stopped early.
- **A finding that does not reproduce is not automatically invalid.** It may be
  environment-specific, already fixed, or dependent on state the tester had and you do
  not. Report which, rather than silently dropping it.
- **Not every row is a bug.** Feature requests and product questions filed as bugs are
  still signal; route them, do not discard them and do not force them into a bug template.
- **The tester is a colleague, not a ticket source.** Evidence problems get reported back
  courteously and specifically, because next round's sheet is better only if this round's
  gaps are named.
- **Sharding the detection-gap lens destroys its best output.** Per-cluster, it produces
  N shallow "add a test here" answers and cannot see the gap that explains several
  findings at once. This is the most expensive mistake available in Phase 3.
- **Two agents agreeing is not verification, and two agents disagreeing usually means
  both are wrong.** Measure contested claims yourself before they reach an issue.
- **The obvious fix sometimes converts a loud failure into a silent one.** A rejected
  write that becomes an accepted no-op is strictly worse than the bug reported. When a
  fix relaxes a validation, check what the code does with the value once it gets through.
- **A safeguard that never runs is a gap, not a safeguard.** Check that the layer you
  are crediting actually executes: suites gated behind tags nothing carries, or guards
  whose missing-input branch exits zero, are green and worthless.
- **"A test exists" is not "the product is tested."** Ask what payload, state, or path
  the test actually exercises, and whether the product ever produces it. A test factory
  that hand-rolls a request tests the factory, not the product — and every hand-rolled
  request tends to be valid by construction, which is exactly why it passes. When a
  defect survived a long time behind green CI, this is the first thing to check.
- **Type-level contract checking is blind to value domains.** A field can be type-valid
  on both sides while the backend requires a narrower set of values than the frontend can
  produce. No amount of shared types catches it; only a shared validator or a test that
  drives the real path does.
- **The oldest bug in the round is the most informative one.** Date each finding's
  introduction. A defect that survived months at high failure rates is telling you
  something about coverage — or about whether anyone uses the feature — that a fresh
  regression cannot.

## Validation

- [ ] The sheet and every referenced artifact were resolved, with a status recorded for each
- [ ] Unreadable or missing evidence was reported rather than skipped
- [ ] Every row was surface-reviewed and appears somewhere in the final output
- [ ] Findings that are not defects were identified as such
- [ ] The tracker was searched for existing issues before any were created
- [ ] Every cluster is justified by a shared file path
- [ ] The investigation mode (full / light) was chosen and stated
- [ ] Lens A ran per cluster; lens B per pattern; lens C **exactly once over all findings**
- [ ] Every agent delivered findings; none was allowed to finish silently
- [ ] Contested claims between agents were measured directly, not averaged or arbitrated
- [ ] Every claim is labelled CONFIRMED or SUSPECTED
- [ ] Every filed issue names a specific detection-gap artifact
- [ ] Repeated detection gaps were called out once, not duplicated per issue
- [ ] Each finding's introduction date was established, and the oldest was investigated
- [ ] Prior rounds' promised artifacts were audited: landed / proposed-only / not-running
- [ ] The report distinguishes artifacts **promised** from artifacts **verified landed**
- [ ] The companion sheet contains a row for every original finding
- [ ] The cycle was appended to the log

## Troubleshooting

**The sheet cannot be read**: confirm the connector has access to that specific file, and
that the identifier is the file itself rather than a folder. Stop rather than proceeding
on partial data.

**Evidence links resolve but the folder listing is short**: the artifacts are shared
per-file rather than folder-wide. Work from the links in the sheet and report the sharing
gap to the tester.

**In-place sheet editing is unavailable**: expected — create the companion sheet instead,
and say so in the report so nobody waits for a column that will not appear.

**A finding cannot be reproduced from the code**: report it as unconfirmed with what you
checked. Do not file a speculative issue, and do not silently drop the row.

**`gh` not authenticated**: run `gh auth status` and resolve before Phase 4. Phases 0-3
are read-only and can run without it.
