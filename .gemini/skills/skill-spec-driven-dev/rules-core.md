# Spec-Driven Development — Always-On Rules (core)

These rules apply to EVERY capability in `skill-spec-driven-dev`. Read once
per session and honour them throughout the task. They are agent-neutral.

Detailed rules — examples, traceability column rules, full host-repo probing
matrix, knowledge-base integration, portability notes — live in
`rules-detail.md` and `references/`. Load them only when a step explicitly
points at them.

---

## 1. Core principles

1. **Specs first, code last.** No production-code change happens before LLD +
   acceptance criteria + failing tests exist for the feature.
2. **Traceability is non-negotiable.** Every TASK traces to an AC; every AC
   traces to a REQ/NFR; every TC traces to an AC; every source change during
   `implement` references ≥1 `TASK-n`.
3. **Graceful degradation, not invention.** Missing / unclear / contradictory
   input → file a `Q-n` in `clarifications.md`. Never silently invent.
4. **Append-only IDs.** IDs are assigned once. Superseded IDs stay with
   `status: retired`. Never renumber.
5. **Progressive disclosure.** Load only what you need for the current step.
   Reference files (`references/*.md`, `rules-detail.md`, the host knowledge
   base) are pulled on demand, not pre-loaded.
6. **One feature at a time** per workspace. Never mix features.

---

## 2. ID scheme

| Prefix   | Meaning                              | Assigned in |
|----------|--------------------------------------|-------------|
| `REQ-n`  | Functional product / feature requirement | `init`  |
| `NFR-n`  | Non-functional requirement            | `init`      |
| `AC-n`   | Acceptance criterion (Gherkin)        | `design`    |
| `LLD-n`  | LLD section (component / interface / data / edge / constraint) | `design` |
| `TASK-n` | Implementation plan task              | `design`    |
| `RISK-n` | Risk / ambiguity / open assumption    | any         |
| `TC-n`   | Test case                             | `tests`     |
| `Q-n`    | Clarification question / blocker      | any         |

Numbering is monotonic per prefix. Continue from the highest existing number
on re-runs. A retired ID keeps its number.

---

## 3. Feature workspace

Default location: `<repo-root>/.specs/<FEATURE_KEY>/`. The user picks both at
`init` time; record the resolved path in `status.json.workspace_path`.

Layout (fixed — do not rename folders or their numeric prefixes):

```
<workspace>/
├── 00-inputs/          inputs.yaml, context-manifest.md, source docs
├── 01-design/          lld.md, acceptance.md, plan.md, risks.md
├── 02-tests/           tests.md, test-matrix.csv
├── 03-impl/            impl-log.md, verify-report.md
├── traceability.md     append-only REQ→AC→LLD→TASK→TC→commit matrix
├── clarifications.md   Q-n entries (open / advisory / resolved)
├── status.json         machine-readable stage / gate state
└── README.md           human overview
```

A capability MUST NOT create files outside the workspace except (a) production
code during `implement`, or (b) a host-repo formal-doc mirror with explicit
user opt-in at `design` time.

---

## 4. Artifacts & frontmatter

Every markdown artifact starts with the standard YAML frontmatter; the full
contract (fields, controlled values, allowed `artifact:` kinds, provenance
header) is in **`references/artifacts.md`**. Load that file when writing or
auditing an artifact.

CSV / JSON artifacts have no frontmatter; they appear in
`status.json.artifacts[]` with a generator stamp.

---

## 5. `status.json` (state machine)

Required fields summary (full schema + example: **`references/status-schema.md`**):

- `feature_key`, `workspace_path`, `title`
- `stage` ∈ `inputs-blocked` | `inputs-ready` | `design-ready` | `tests-ready` | `implement-done`
- `stages.{init,design,tests,implement,lint}.{done,at,gates,...}`
- `counters.{REQ,NFR,AC,LLD,TASK,TC,RISK,Q}` — must match actual counts on disk
- `open_clarifications`, `open_risks`
- `toolchain`, `host_integrations`, `artifacts[]`

`stage` is monotone unless a capability is intentionally re-run, which reverts
`stage` to that capability's "ready" value. Every capability updates
`status.json` atomically at the end of its run.

---

## 6. Traceability

`traceability.md` is the single source of truth for cross-artifact links —
append-only Markdown table:

```
| REQ/NFR | AC | LLD | TASK | TC | Commits | Status |
```

Required invariants (lint enforces):
- Every REQ/NFR, AC, TASK, and TC appears in ≥1 row.
- Empty cells are `—` (never blank).
- Rows are added, never silently modified. Changed relationships → new row,
  mark prior row `status: superseded`.

Detailed column rules and `status` enum: `rules-detail.md` §traceability.

---

## 7. Clarifications & ambiguity

`clarifications.md` is the running log of `Q-n` entries. Severity drives
behaviour:

| Severity   | Behaviour                                                |
|------------|----------------------------------------------------------|
| blocker    | STOP the current capability. Do not advance `stage`.     |
| advisory   | Continue. Record assumption inline with ⚠️. Revisit.      |
| resolved   | Keep entry, mark resolved with date.                     |

Never silently reconcile contradictory sources. File a `Q-n` instead.

Entry template, source-anchoring conventions, and rollover rules: see
`rules-detail.md` §clarifications.

---

## 8. Confidence & verification

- `high` = supported by a cited primary source AND human-reviewed (AI-only
  verification does NOT count).
- `medium` = AI-extracted, not yet human-reviewed (default for fresh output).
- `low` = inferred from indirect signals; mark with ⚠️ inline.

Cite sources with `(source: <file_or_url>#anchor)` for every non-trivial
claim. Prefer primary sources.

---

## 9. Security & safety (hard guarantees)

- **Never auto-commit.** `implement` leaves the working tree dirty for human
  review.
- **Never modify unrelated files.** `implement` may only touch files listed
  in the current `TASK-n.files` (or new files explicitly added by the design).
- **Respect ignore lists** — `.gitignore`, `.claudeignore`, `.cursorignore`,
  `.clineignore`, and any custom ignore files in the host repo.
- **Never transmit workspace content / source / credentials** outside the
  user's local environment unless the user explicitly asks.
- **Flag, don't redact** if a secret is detected — prompt the user before
  persisting.

---

## 10. Self-validation gate

Every capability must self-validate before exit. The protocol is in
**`references/self-validation.md`**; each capability lists its own 2–3
questions in its "self-validation" section.

---

## 11. Portability

- Use generic verbs (*read the file*, *run the build*). No agent-specific
  tool names.
- Refer to external systems by capability (e.g. "fetch the Jira ticket") and
  always offer a manual paste fallback.
- All state lives on disk under the workspace. Any agent (or human) may
  resume from any stage by reading `status.json`.
- Plain CommonMark + YAML only. No proprietary markdown extensions, no
  executable code blocks.

---

## 12. Update protocol

When this rules file changes, bump `version` in `SKILL.md` and add a row to
the changelog in `rules-detail.md` §changelog.
