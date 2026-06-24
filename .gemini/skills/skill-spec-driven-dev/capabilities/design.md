# Capability: design — Low-level design + acceptance criteria + plan

Produce the three design artifacts that everything downstream depends on:
LLD, acceptance criteria (AC), and an ordered implementation plan (TASK
list). Also produce / update the risk register.

## Preconditions

- `status.json.stage` ∈ { `inputs-ready`, `design-ready` }  (re-runnable)
- `00-inputs/inputs.yaml` exists with ≥1 REQ or NFR
- No `Q-n` of severity `blocker` unresolved
- If any blocker exists, STOP and tell the user to resolve clarifications.

## Read order (selective)

1. `<workspace>/00-inputs/inputs.yaml`
2. `<workspace>/00-inputs/context-manifest.md`
3. Source docs under `<workspace>/00-inputs/*.md` (pm, hld, jira,
   architecture-refs, extras)
4. If `inputs.yaml.ui_inputs.applicable: true`: also read
   `00-inputs/ui-prototype.md`, `00-inputs/figma-specs.md`, every image in
   `00-inputs/screenshots/` (or the URL list in `screenshots.md`). Use these
   to inform the LLD's UI/UX section.
5. If host repo has a knowledge base: its schema/index, then only pages in
   scope (target modules, concepts, patterns, protocols). KB integration
   guidance: **`rules-detail.md` §knowledge-base**.
6. If host repo has a design-doc folder: module-level overview(s) for the
   target area and any directly relevant ADRs.
7. Source code: only headers / interface files of modules referenced by
   requirements (do NOT read the whole codebase).

## Outputs

- `01-design/lld.md`
- `01-design/acceptance.md`
- `01-design/plan.md`
- `01-design/risks.md`
- Updates: `traceability.md`, `clarifications.md`, `status.json`,
  workspace `README.md` (links section).

Optional, with explicit user opt-in: mirror the LLD into a host-repo
formal-doc folder. Ask before doing this.

## Steps

### 1. Decide scope

- Identify target subsystems / modules / packages from REQs, NFRs, and any
  knowledge-base pages. List at the top of `lld.md`.
- Identify affected build targets / platforms (if multi-target).
- If scope cannot be determined, file a blocker `Q-n` and stop.

### 2. Draft the LLD → `01-design/lld.md`

Use `../templates/lld.md` as the skeleton. Every section gets an `LLD-n` id
and a `traces:` line linking to `REQ-*` / `NFR-*`. Delete any section that
has no real content (do not leave "TBD").

Required sections (omit only if truly N/A):

1. **Overview & Scope** — one paragraph. What this does, what it doesn't do.
2. **Context & Assumptions** — key assumptions; each unconfirmed assumption
   has a matching `Q-n`.
3. **Affected components / modules / packages** — table.
4. **Interfaces** — public API, function signatures, ICD commands, wire
   formats, REST / gRPC / event payloads, CLI flags, headers exported. Each
   interface item gets its own `LLD-n`.
5. **Data model** — structs / classes / typedefs / DB / message schemas.
   Lifetime, ownership, thread-safety for each. Each entity gets its own
   `LLD-n`.
6. **State & concurrency** — state machines, threading model, locks, async
   boundaries, re-entrancy, idempotency.
7. **Dependencies** — internal modules, 3rd-party libs, services. Pinned
   versions where relevant.
8. **Error handling & observability** — return codes / exceptions, error
   taxonomy, logging, metrics, tracing, alerting hooks.
9. **Configuration** — compile-time flags, runtime config keys / env vars /
   CLI / feature flags.
10. **UI / UX** — REQUIRED when `ui_inputs.applicable: true`. Include the
    seven `LLD-UI-*` sub-sections from `templates/lld.md` (screens/routes,
    component decomposition, state & props contracts, accessibility,
    responsive design, design tokens / theming, i18n / l10n). Reference
    source visuals from `00-inputs/`. Skip entirely when not applicable.
11. **Edge cases** — explicit bullet list. Each edge case tagged with the
    `AC-n` that will cover it. See **`references/risk-categories.md`** §2
    for edge-case prompts; pick the categories that apply.
12. **Constraints** — performance, memory, latency, throughput, security,
    certification, accessibility, licensing.
13. **Backward / forward compatibility** — wire / ABI / persisted-data
    compatibility, feature-flag plan, migration strategy.
14. **Rollout & fallback** — feature flag, gradual rollout, rollback.
15. **Open questions** — bullet list of all related `Q-n` ids.

Rules:
- Write for a reviewer, not the compiler. Tables + diagrams-as-code sparingly
  (Mermaid only if helpful).
- No code block longer than ~20 lines. Link to source paths instead.
- Mark every claim ✅ verified / ⚠️ inferred.
- Respect the host repo's knowledge-base conventions if one is present.

### 3. Draft acceptance criteria → `01-design/acceptance.md`

Use `../templates/acceptance.md`. One `AC-n` per criterion. Gherkin form:

```
## AC-1  (REQ-1, NFR-1)
Given <precondition>
When  <action>
Then  <expected observable outcome>
Notes: <measurement technique, tolerances>
```

Coverage rules:
- Every `REQ-*` and `NFR-*` MUST be covered by ≥1 `AC-n`.
- Unclear or non-testable NFRs get an explicit `AC-n` of kind `manual` plus
  a `Q-n` noting the gap.
- Negative / error-path criteria count.
- Cross-cutting NFRs get explicit measurable criteria, not "should be fast".

### 4. Draft implementation plan → `01-design/plan.md`

Use `../templates/plan.md`. Ordered `TASK-n`s grouped by phase:

1. **Scaffold** — skeletons, new files, empty stubs, build-system wiring.
2. **Core** — primary functionality.
3. **Integration** — wiring into existing subsystems, callers, adapters.
4. **Hardening** — error paths, edge cases, observability, docs.

Each TASK:

```yaml
- id: TASK-1
  title: Short imperative ("Add ML-KEM-1024 keygen handler")
  description: |
    What changes, why, at a glance.
  depends_on: []
  lld_refs:  [LLD-3, LLD-7]
  ac_refs:   [AC-2]
  risk_refs: [RISK-1]
  files:     [firmware_vob/.../cry_mlkem.c, firmware_vob/.../cry_mlkem.h]
  validation:
    - build: "<build command>"
    - tests: [TC-? placeholder, filled in by /spec-tests]
  est_effort: S          # S | M | L
```

Rules:
- Order respects `depends_on`; a task cannot precede its dependency.
- No task exceeds ~1 day of work; split if too large.
- Every `AC-n` has ≥1 task that will make its test green (filled in
  post-`tests`).
- `files:` must be specific paths; no globs, no "tbd".

### 5. Populate risk register → `01-design/risks.md`

Use `../templates/risks.md`. One `RISK-n` per significant risk:

```
## RISK-1
- Description: ...
- Likelihood: low | medium | high
- Impact: low | medium | high
- Mitigation: ...
- Owner: ...
- Related: LLD-?, TASK-?, Q-?
```

Walk the categories in **`references/risk-categories.md`** §1 and pick the
ones that apply. Don't invent risks; record only those with genuine
likelihood.

### 6. Update `traceability.md`

Append one row per new relationship. Every REQ/NFR, AC, LLD, TASK appears
in ≥1 row. Leave `TC` and `Commits` as `—`. Column rules:
**`rules-detail.md` §traceability**.

### 7. Optional mirror into host-repo formal-doc folder

If the host repo has a formal-docs convention, ASK:

> "Should the LLD also be written to `docs/<...>/<feature>.md` to match your
> team's formal-doc convention? (y/n)"

If yes, write a trimmed version there (overview, interfaces, data model,
diagrams if any) and add a `See also: <path>` backlink in
`01-design/lld.md`. Never duplicate full content in both places without a
backlink.

### 8. Update `status.json`

Update per **`references/status-schema.md`** §3:
- `stage`: `design-ready` if all gates pass (see below). Otherwise keep
  prior stage and record which gate failed in `stages.design.gates`.
- `stages.design.done` = `true` with ISO-8601 timestamp on success.
- Increment counters (`AC`, `LLD`, `TASK`, `RISK`).

### 9. Self-validation

Run the protocol in **`references/self-validation.md`** with:

1. "For requirement REQ-1, which AC covers it, and which TASK implements
   it?" → `traceability.md`.
2. "What does component X expose externally?" → `lld.md` §Interfaces.
3. "What are the top 3 risks and their mitigations?" → `risks.md`.
4. "List every assumption made because of a missing input." → `lld.md`
   §Context & Assumptions + `clarifications.md`.

### 10. Report to the user

Print a short summary:
- Counts: REQ/NFR, AC (and coverage %), LLD sections, TASK count per phase,
  RISK count.
- List any open advisory `Q-n`.
- If any gate failed, say so explicitly and stop.
- If all gates passed, invite the user to run `tests` next.

## Gates (must all pass to advance `stage` to `design-ready`)

- `lld_complete` — every required section is present or explicitly marked
  `N/A` with a one-line justification.
- `acceptance_covers_all_reqs` — every REQ and NFR appears in ≥1 AC row in
  `traceability.md`.
- `plan_ordered_and_sized` — `depends_on` forms a DAG (no cycles), every AC
  has ≥1 TASK, no TASK is effort `L` without justification.
- `risks_registered` — every blocker `Q-n` has a matching `RISK-n` OR is
  resolved.
