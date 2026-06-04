---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "pr-review",
  "title": "PR Review",
  "version": "1.0.0",
  "status": "Draft",
  "risk_level": "low",
  "requires_approval": false,
  "objective": "Multi-dimensional review of a PR diff across security, architecture, code quality, performance, and accessibility — run one lens or all of them",
  "description": "Review the current PR diff across one or more dimensions: security (OWASP), architecture (boundaries & design), code quality (DRY, type safety, conventions), performance (rendering, queries, bundle), and accessibility (WCAG 2.1 AA). Use when user says 'review PR', 'pr review', 'security review', 'architecture review', 'code quality review', 'performance review', 'accessibility review', or '/pr-review'.",
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "category": [
    "review"
  ],
  "tags": [
    "pr-review",
    "security",
    "architecture",
    "code-quality",
    "performance",
    "accessibility",
    "owasp",
    "wcag",
    "skill"
  ],
  "inputs": {
    "optional": [
      {
        "name": "dimension",
        "description": "Which review lens(es) to run: security | architecture | quality | performance | accessibility | all. Accepts a comma-separated list. Defaults to all.",
        "type": "string",
        "default": "all",
        "example": "security,performance"
      },
      {
        "name": "target",
        "description": "PR number to review. If omitted, reviews the current branch diff against the default branch.",
        "type": "string",
        "default": "",
        "example": "1640"
      },
      {
        "name": "base_branch",
        "description": "Base branch to diff against when no PR number is given.",
        "type": "string",
        "default": "main",
        "example": "develop"
      }
    ]
  },
  "checksum": {
    "algorithm": "sha256",
    "hash": "b4e4ca526bd697410a68dd6a12501ab4cc6202ee5d3df92004fc0709bc4c11e0"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "ZI0Xr6PcnOAlt8bzdUShPpbC3AIdfhagYMk7OSv4MrY30WRXWs/n9g2UrFdKUZLhGfWzD9hd3V4FAuzu6qtzBQ==",
    "public_key": "-----BEGIN PUBLIC KEY-----\nMCowBQYDK2VwAyEAm97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=\n-----END PUBLIC KEY-----\n",
    "signed_at": "2026-06-04T11:00:07.348Z",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# PR Review

A single multi-dimensional reviewer for a PR diff. Pick one lens or run them all. The security and architecture lenses are stack-agnostic; the quality, performance, and accessibility lenses are tuned for the imboard stack (TypeScript monorepo — React + MUI Joy UI + Vite frontend, Express + MongoDB/Mongoose backend, shared-types package).

## Trigger

Run when the user says any of: "review PR", "pr review", "/pr-review", or a single-dimension phrase — "security review", "architecture review" / "design review", "code quality review" / "code review", "performance review", "accessibility review" / "a11y review".

A single-dimension phrase implies `dimension` = that lens. A bare "review PR" implies `dimension=all`.

## Step 0 — Resolve scope

Parse the request:

- **`dimension`** — map the trigger phrase (or an explicit flag) to one or more of: `security`, `architecture`, `quality`, `performance`, `accessibility`. "review PR" with no qualifier → run **all five**.
- **`target`** — a PR number if one was given, else empty.
- **`base_branch`** — default `main`.

## Step 1 — Get the diff (shared)

Fetch the changes once and reuse them for every selected dimension:

- If a PR number is provided: `gh pr diff <target>`
- Otherwise: `git diff <base_branch>...HEAD`

Note which files changed and their types (backend `.ts`, frontend `.tsx`, config, etc.) so each lens only inspects relevant files.

## Step 2 — Run the selected dimension(s)

For each selected dimension, work through its checklist below. Only flag **real** issues with specific `file:line` references — no generic advice. If a dimension finds nothing, say so explicitly.

---

### Dimension: security

Stack-agnostic security review against the OWASP Top 10. Audit every changed file:

**Injection & data access**
- **SQL / NoSQL injection**: user input (`req.body` / `req.query` / `req.params`, route params, headers) reaching a query without parameterization or sanitization. For document stores, watch for operator injection (e.g. `$where`, `$regex`, user-controlled query objects).
- **Command / path traversal**: user input reaching shell execution or `fs` operations. Check that decoded paths are validated against a safe root before use.
- **Mass assignment**: update operations that spread an unsanitized request body directly into the persisted object — allowlist the fields instead.

**AuthN / AuthZ**
- **Missing authentication**: new routes/endpoints that skip the auth middleware they should inherit.
- **Broken object-level authorization (IDOR)**: can a user access another tenant's/owner's resource by changing an ID in the path or body? Verify ownership/scope is checked server-side, not assumed.
- **Privilege / entitlement checks**: gated functionality (paid, admin, role-restricted) reachable without the corresponding guard.

**Web surface**
- **XSS**: `dangerouslySetInnerHTML` / `innerHTML` (or equivalent) fed user-generated content without sanitization.
- **Open redirects**: navigation or `Location` headers built from user-controlled URLs.
- **SSRF**: server-side fetches of user-supplied URLs without an allowlist.
- **Token storage**: auth tokens in `localStorage`/JS-readable storage without justification; prefer httpOnly cookies.
- **CORS**: changes that broaden allowed origins beyond what's needed.

**Integrations & secrets**
- **Webhook authenticity**: inbound webhooks must verify their signature over the **raw** request body (not the re-serialized JSON).
- **Secret exposure**: secrets in code, logs, or error responses; tokens/keys leaking into client-facing payloads.
- **Rate limiting**: new public endpoints (login, signup, password reset, webhooks) without throttling.

**Dependencies & errors**
- **Vulnerable dependencies**: newly added packages — check for known CVEs.
- **Error information leakage**: stack traces, internal paths, or schema details surfaced in responses.

---

### Dimension: architecture

Stack-agnostic review of structure and design. Audit changed files for:

**Module boundaries**
- **Boundary violations**: layers/packages importing across boundaries they shouldn't (e.g. a UI layer reaching into server internals, or two sibling packages depending on each other instead of through a shared contract).
- **Dependency direction**: shared/contract packages must not depend on the apps that consume them; dependencies point one way.
- **Shared contracts**: types/interfaces used by more than one consumer should live in the shared module, not be duplicated.

**Separation of concerns**
- **Service layer**: business logic belongs in service/domain modules, not in route handlers or controllers. Handlers parse input, call a service, format output.
- **Decoupled collection vs presentation**: data-gathering should be separable from formatting/serialization so one can change without the other.
- **Pure functions for logic**: stateless business rules extracted into pure, dependency-free functions are easier to test and reuse — flag logic entangled with I/O that didn't need to be.
- **Middleware placement**: route-specific middleware belongs on the route definition (where it can see route params), not registered globally.

**Data flow**
- **Unidirectional flow**: data flows down, events flow up. Flag bidirectional coupling between components/modules.
- **Contract consistency**: a change to a request/response shape must be reflected everywhere it's produced and consumed (server handler, client caller, shared types).
- **Single source of truth**: derived state (billing, permissions, totals) should be computed in one place and flow outward, not recomputed via parallel paths.

**Design & scale**
- **Single responsibility**: modules that mix data access + business logic + formatting are a smell.
- **Open/closed**: can the change be extended without editing existing code? Especially for plug-in-style sets (block types, states, notification kinds).
- **Schema changes**: new fields/collections — consider migration of existing data, query-performance, and storage growth.
- **Endpoint & job design**: new endpoints that will need pagination/filtering as data grows; long-running work that belongs in a background job rather than a request handler.

---

### Dimension: quality

Code-quality review tuned for the imboard TypeScript monorepo (`packages/frontend`, `packages/backend`, `packages/shared-types`):

**TypeScript type safety**
- **`any` usage**: new `any` without justification — prefer `unknown`, a specific type, or a generic.
- **Unsafe assertions**: `as` casts that bypass checking without runtime validation (especially on API response data).
- **Missing return types**: exported functions/methods without explicit return types (internal functions may rely on inference).
- **Shared-types pattern**: types used by both frontend and backend that are duplicated instead of living in `shared-types`. The pattern is `const Foo = { key: { label, description } } as const; type FooType = keyof typeof Foo;`.
- **Non-null assertions**: `!` without a preceding guard — prefer `?.` or an explicit check.

**Error handling**
- **Swallowed errors**: `catch {}` that silently drops errors. Empty catch is acceptable per project convention only when the error genuinely doesn't matter.
- **Unhandled rejections**: `async` calls without `try/catch` or `.catch()`, especially in event handlers and `useEffect`.
- **Express error flow**: handlers that throw but don't pass the error to `next()`.
- **Generic user-facing errors**: "Something went wrong" without actionable guidance.

**DRY & maintainability**
- **Duplication**: repeated logic that should be a shared util/hook.
- **Magic values**: hardcoded numbers/strings (trial durations, retry counts, status strings) that should be named constants.
- **Long functions / deep nesting**: functions > ~50 lines or > 3 levels of nesting — prefer early returns and extraction.
- **Boolean params**: flag arguments that make call sites unclear — prefer an options object or distinct functions.

**imboard conventions**
- **Phosphor icons**: import from `@phosphor-icons/react/dist/ssr/<IconName>`; the `Icon` type for maps comes from `@phosphor-icons/react`.
- **Recharts**: must be wrapped in `<NoSSR>`.
- **Billing**: derive billing state via `computeBillingSummary()` from `billingService.ts` — never recompute it inline.
- **Unused vars**: prefix with `_`; remove unused imports entirely.
- **Catch binding**: use `catch {` when the error isn't used (no unused binding).
- **Hooks before returns**: `useMemo`/`useCallback` must precede any early return (rules-of-hooks).
- **Ref cleanup**: capture `ref.current` in a local inside `useEffect` and use the local in cleanup.
- **ESM imports**: no `.js` extensions (project uses `moduleResolution: "Bundler"`).

**API & testing**
- **Input validation**: handlers validate `req.body`/`req.params`/`req.query` before use.
- **Status codes**: 201 create, 204 delete, 400 bad input, 404 not found, 409 conflict.
- **Missing tests**: new pure business-logic functions without tests; tests that depend on execution order; backend pure-function tests that import heavy modules (mongoose/full app) and risk OOM — import from lightweight modules instead.

---

### Dimension: performance

Performance review tuned for the imboard stack (React + MUI Joy UI + Vite frontend; Express + MongoDB/Mongoose backend):

**Frontend (React + Joy UI + Vite)**
- **Unnecessary re-renders**: new object/array/function literals passed as props each render (`style={{…}}`, `options={[…]}`, `onClick={() => fn(x)}`) without `useMemo`/`useCallback`.
- **Missing memoization**: expensive work in the render path (chart data shaping, editor init, member-list processing) without `useMemo`.
- **Heavy imports**: `import *` / barrel imports that defeat tree-shaking. Phosphor icons must use the `@phosphor-icons/react/dist/ssr/<IconName>` path.
- **Bundle size**: new dependencies — estimate gzipped size; flag anything > ~50KB.
- **Code splitting**: new route-level components not using `React.lazy()`.
- **List rendering**: large lists without virtualization; index keys where items can reorder.
- **Effect deps**: `useEffect` with missing or overly-broad dependency arrays (infinite loops / stale closures).
- **Recharts**: must be wrapped in `<NoSSR>`.

**Backend (Express + MongoDB/Mongoose)**
- **N+1 queries**: per-item queries inside a loop instead of `$in` / `aggregate` / `populate`.
- **Missing indexes**: new `find`/`findOne`/`aggregate` query shapes on unindexed fields — indexes are declared in the schema.
- **Unbounded queries**: `find({})` without `.limit()` / pagination.
- **Missing `.lean()`**: read-only queries that don't need Mongoose documents.
- **Aggregation shape**: `$lookup` without supporting indexes; `$match` not placed before `$lookup`.
- **Blocking the event loop**: sync file I/O, CPU-heavy work, or `JSON.stringify` of large objects in a request handler.

**Shared**
- **Payload size**: responses returning more than the client needs — project query fields.
- **Caching**: repeated identical queries/computations that could be memoized (e.g. follow the pure-function pattern of `computeBillingSummary`).
- **Parallelism**: sequential `await`s that could be `Promise.all()`.

---

### Dimension: accessibility

WCAG 2.1 Level AA review tuned for the imboard frontend (React + MUI Joy UI, BlockNote editor, Recharts). Audit changed UI files:

**Joy UI components**
- **Missing labels**: `Input`/`Textarea`/`Select`/`Autocomplete` without an associated `FormLabel` or `aria-label` (use the `FormControl` + `FormLabel` pattern).
- **Icon-only buttons**: `IconButton` without an `aria-label` — a Phosphor icon alone is not a label.
- **Modal / Drawer**: `Modal` needs `aria-labelledby` (title) and `aria-describedby`; focus must be trapped. `Drawer` must move focus in on open and restore it on close.
- **Alert / Snackbar**: custom alerts need `role="alert"`; auto-dismiss ≥ 5s or configurable.
- **Color-only meaning**: status conveyed only by `color` (error/success) without text or icon.

**Keyboard**
- **Tab order**: logical order; avoid `tabIndex > 0`; don't make focusable elements `tabIndex={-1}`.
- **Focus visible**: custom-styled interactives must keep a visible focus ring (no `outline: none` without a replacement).
- **Non-button handlers**: `onClick` on `div`/`span` without matching `onKeyDown` for Enter/Space.
- **Focus management**: route changes, dynamic insertion, and modal open/close must move focus deliberately.

**BlockNote & charts**
- **Custom blocks / toolbar / slash menu**: need accessible labels and keyboard reachability.
- **Charts**: Recharts needs a text alternative (summary table or `aria-label`); chart colors (`var(--joy-palette-*)`) must meet contrast; tooltips must be keyboard-accessible.

**Media, forms, dynamic content**
- **Alt text**: meaningful `alt` on images/`Avatar`; `alt=""` only for decorative; inline SVGs need `role="img"` + label or `aria-hidden`.
- **Form errors**: validation errors associated via `aria-describedby` / `FormHelperText`; `aria-invalid` on the field; required fields marked `required`/`aria-required`.
- **Live regions**: dynamic updates (notifications, status changes) need `aria-live`; loading states need `aria-busy` + `role="status"`.

Reference the specific WCAG criterion when flagging (e.g. "1.4.3 Contrast (Minimum)", "2.1.1 Keyboard"). Joy UI handles many concerns by default — only flag where defaults are overridden or custom components are used.

---

## Step 3 — Output (shared)

Emit one section per dimension you ran, each using this severity structure:

```
## <Dimension> Review

### Critical
- [FINDING]: description + file:line + fix

### High
- [FINDING]: description + file:line + fix

### Medium
- [FINDING]: description + file:line + fix

### Low / Informational
- [FINDING]: description + file:line + suggestion

### Clean
- areas reviewed and found sound
```

If a dimension is clean, say so explicitly (e.g. "No security issues found in this diff") with a one-line summary of what was checked.

When multiple dimensions run, end with a short **Summary** line: counts per severity across all dimensions and the single most important thing to fix first.

## Important

- Flag only real issues with concrete `file:line` references — no generic advice.
- Distinguish "must fix" (Critical/High) from "nice to have" (Medium/Low).
- Acknowledge good patterns when you see them — positive signal matters.
- Zero-tolerance: if you spot pre-existing issues in the changed files, flag them too rather than scoping them out.
- Pragmatism over purity: only raise architecture/design issues that cause real pain (maintenance, bugs, scaling), not theoretical concerns.
