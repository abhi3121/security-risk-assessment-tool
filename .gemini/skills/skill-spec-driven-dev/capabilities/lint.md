# Capability: lint — Traceability & coverage health check

Verify the feature workspace is internally consistent: every requirement has
acceptance criteria, every AC has tests, every task will turn at least one
test green, every TC maps back to a requirement, and `status.json` reflects
reality. Produces a report; does not modify other artifact content.

## Preconditions

- A workspace exists at `status.json.workspace_path`.
- Any stage ≥ `inputs-ready` is acceptable — lint runs at whatever stage
  the feature is in and flags what isn't yet possible to verify.

## Read order

1. `status.json`
2. `00-inputs/inputs.yaml`
3. `01-design/{lld.md, acceptance.md, plan.md, risks.md}` (if present)
4. `02-tests/{tests.md, test-matrix.csv}` (if present)
5. `03-impl/{impl-log.md, verify-report.md}` (if present)
6. `traceability.md`
7. `clarifications.md`

## Outputs

- `lint-report-YYYY-MM-DD.md` in the workspace root.
- Update `status.json.stages.lint` with the run timestamp and a `pass`
  boolean.

## Checks

For each finding, record: severity (`error` / `warn` / `info`), location,
and suggested fix. Run only the checks applicable to the current stage.

### A. Requirement coverage

- A1. Every `REQ-*` in `inputs.yaml` appears in ≥1 row of `traceability.md`.
- A2. Every `NFR-*` in `inputs.yaml` appears in ≥1 row of `traceability.md`.
- A3. Every `REQ/NFR` transitively reaches at least one `AC` (if stage ≥
  `design-ready`) and one `TC` (if stage ≥ `tests-ready`).

### B. Acceptance criteria

- B1. Every `AC-n` in `acceptance.md` appears in ≥1 row.
- B2. Every `AC-n` has ≥1 `TC` (automated or `manual`), if stage ≥
  `tests-ready`.
- B3. No `AC-n` is defined without a preceding or matching `REQ/NFR`.

### C. Design

- C1. Every `LLD-n` in `lld.md` has a `traces:` line and every id referenced
  there exists.
- C2. Every design section marked `⚠️` is also referenced from a `Q-n` or a
  `RISK-n`.

### D. Plan & tasks

- D1. Every `TASK-n` has non-empty `lld_refs`, `ac_refs`, and `files`.
- D2. `depends_on` graph is a DAG (no cycles).
- D3. Every `AC-n` has ≥1 `TASK-n` whose `ac_refs` includes it (if stage ≥
  `design-ready`).
- D4. If stage ≥ `tests-ready`, every `TASK-n` has non-empty
  `validation.tests`.
- D5. If stage ≥ `implement-done`, every `TASK-n` is `status: done` or
  `skipped` (with justification in `impl-log.md`).

### E. Tests

- E1. `test-matrix.csv` is well-formed: stable header, one row per TC, no
  duplicate TC ids.
- E2. Every `TC-n` in `tests.md` appears in `test-matrix.csv` and vice versa.
- E3. If stage = `tests-ready`, every automated `TC` has `status: red`.
- E4. If stage = `implement-done`, every automated `TC` has `status: green`.
- E5. Every `TC` `location` points to a file that exists on disk (if a
  skeleton was generated).

### F. Clarifications & risks

- F1. No `Q-n` of severity `blocker` is unresolved.
- F2. Every advisory `Q-n` older than 14 days is flagged for review.
- F3. Every `RISK-n` has a mitigation.
- F4. Every `Q-n` referenced in an artifact exists in `clarifications.md`.

### G. Traceability matrix integrity

- G1. No duplicate rows (same REQ+AC+TASK+TC combination).
- G2. Empty cells are `—`, not blank.
- G3. `status` values are from the valid enum (see **`rules-detail.md`
  §traceability**).
- G4. `Commits` column is `—` until `implement-done`, then placeholder
  `pending` or an actual SHA.

### H. `status.json` integrity

- H1. File parses.
- H2. `counters` matches actual artifact counts (±0).
- H3. `stage` is monotone (no backward jumps unless a capability re-run
  intentionally reset it).
- H4. `artifacts[]` lists every file under the workspace.

### I. Filesystem hygiene

- I1. No orphan files in the workspace (anything written not listed in
  `artifacts[]`).
- I2. No accidentally-committed secrets pattern in any artifact (basic
  regex sweep: `API_KEY=`, `PRIVATE KEY`, `password=`, AWS keys, etc.).
  Flag; do not redact automatically.
- I3. Workspace files respect the host repo's ignore lists.

### J. (If knowledge base present) host-wiki alignment

- J1. If an `implement` stage completed and touched a module with a wiki
  page, confirm that page's `last_updated` is today's date (or the
  host-schema equivalent).
- J2. If the host schema defines a changelog, confirm the feature key
  appears in it.

### K. (If `ui_inputs.applicable: true`) UI coverage — all `info`

- K1. Every screen / route declared in `lld.md` `LLD-UI-1` has ≥1 `TC`.
- K2. Every `LLD-UI-*` sub-section that is present has a `traces:` line.
- K3. If `00-inputs/screenshots/` exists, every image referenced from
  `lld.md` resolves to an actual file (or to a URL in `screenshots.md`).
- K4. When UI is in scope, at least one accessibility-level TC exists in
  `tests.md` (description mentions axe / a11y / WCAG).

K-series checks emit `info` rather than `error` so teams can opt in to UI
rigour incrementally.

---

## Report format → `lint-report-YYYY-MM-DD.md`

Frontmatter and naming per **`references/artifacts.md`**. Body skeleton:

```markdown
# Lint report — <feature_key> — YYYY-MM-DD

**Overall result:** PASS | FAIL (N errors, M warnings, K infos)

## Summary by category

| Category | Errors | Warnings | Infos |
|---|---|---|---|
| A Requirement coverage | … | … | … |
| B Acceptance criteria | … | … | … |
| C Design | … | … | … |
| D Plan & tasks | … | … | … |
| E Tests | … | … | … |
| F Clarifications & risks | … | … | … |
| G Traceability matrix | … | … | … |
| H status.json integrity | … | … | … |
| I Filesystem hygiene | … | … | … |
| J Knowledge-base alignment | … | … | … |
| K UI coverage (if applicable) | … | … | … |

## Findings

### A1  [error]  REQ-3 has no row in traceability.md
- **Location:** traceability.md
- **Suggested fix:** add a row `| REQ-3 | — | — | — | — | — | needs-design |`.

### E3  [warn]  TC-7 is status=red but no TASK targets it
- **Location:** test-matrix.csv:8 / plan.md
- **Suggested fix:** extend TASK-4.validation.tests with TC-7.

...
```

End with `## Next actions` listing, in priority order, what the user (or a
future agent run) should do to bring the feature to a healthy state.

## `status.json` update

```json
"stages": {
  "lint": {
    "done": true,
    "at": "2026-05-06T18:00:00Z",
    "pass": false,
    "errors": 2,
    "warnings": 5,
    "infos": 3,
    "report": "lint-report-2026-05-06.md"
  }
}
```

## Report to the user

- Print the summary table and `Next actions` list inline.
- Point to the full report path.
- If `pass: false` and the next stage's gates would fail, explicitly say
  "`<stage>` capability cannot advance until these errors are fixed".

Lint never changes other artifacts; the user or another capability run is
responsible for remediation.
