# Capability: implement — TDD implementation, task-by-task

Turn every failing test green by implementing tasks in order, respecting
dependencies, running the test target after each task, and iterating on
failures. **Never auto-commit;** leave the working tree dirty for human
review (full safety rules in `rules-core.md` §9).

## Preconditions

- `status.json.stage` = `tests-ready`
- `status.json.stages.tests.gates.red_state_confirmed` = `true`
- No `Q-n` of severity `blocker` unresolved

If preconditions fail, STOP and tell the user which earlier capability to
(re-)run.

## Read order

1. `01-design/plan.md` — ordered TASK list
2. `02-tests/tests.md` and `02-tests/test-matrix.csv` — what must turn green
3. `01-design/lld.md` — the design contract to implement against
4. `01-design/acceptance.md` — to keep AC intent in view
5. `01-design/risks.md` — extra care around flagged areas
6. `status.json.toolchain` — for build + test commands
7. If a knowledge base exists, pages for modules being modified (see
   **`rules-detail.md` §knowledge-base**).

## Outputs

- Source-code changes under the host repo (only inside the `files:`
  declared by the current TASK, or new files explicitly introduced by the
  design).
- `03-impl/impl-log.md` — ordered TASK journal.
- `03-impl/verify-report.md` — final build/test report on finalisation.
- Knowledge-base / design-doc updates where the change affects documented
  behaviour (follow host conventions; otherwise leave new content
  `confidence: medium`).
- Updates: `traceability.md` (Commits column), `status.json`, `plan.md`
  (task status), `clarifications.md` (if new questions arise).

## Core loop (per TASK, in `plan.md` order respecting `depends_on`)

For each `TASK-n`:

### 1. Preflight

- If any dependency in `TASK.depends_on` is not `status: done` in
  `plan.md`, stop the loop and report — plan ordering has been violated.
- Re-read the TASK's `lld_refs`, `ac_refs`, `risk_refs`, `files`,
  `validation.tests`.

### 2. Record intent → `impl-log.md`

```markdown
## TASK-n — started YYYY-MM-DDTHH:MM:SSZ
- Goal: <one sentence>
- Must turn green: TC-a, TC-b, ...
- Intended file changes: <files from plan.md>
- Relevant LLD sections: LLD-x, LLD-y
- Notes / assumptions: ...
```

### 3. Implement

- Modify ONLY files listed in the TASK (plus obvious paired support files —
  e.g. headers paired with source). If a new file outside the declared set
  becomes necessary, STOP, add a `scope-deviation` note to `impl-log.md`,
  and ask the user to confirm.
- Follow host-repo style (formatter, linter, naming).
- Do not touch unrelated code or opportunistically refactor.
- Respect ignore lists per `rules-core.md` §9.

### 4. Build

Run `status.json.toolchain.build_command`. On failure:
- Capture error output, append to `impl-log.md`.
- Fix build errors first (they block all tests).
- Count this as attempt 1 of max 3 for this TASK.

### 5. Run the relevant tests

Prefer the narrowest scope that proves the TASK:
- Start with `validation.tests` (just those TCs).
- On pass, expand to all TCs that reference any `LLD-n` this TASK touches
  (to catch in-feature regressions).
- Record result in `impl-log.md`.

### 6. Iterate on failure (bounded, max 3 attempts)

- If the **test** is wrong (misread spec, typo, bad fixture), fix the test
  AND log under `### Test fixes` in `impl-log.md` with a one-line
  justification. Do NOT simply delete failing assertions.
- If the **production code** is wrong, fix it.
- If the **spec** is wrong or ambiguous, STOP, file a new `Q-n` (blocker if
  it blocks progress; advisory otherwise), do not advance the TASK.

After 3 unsuccessful attempts:
- Mark TASK `status: blocked` in `plan.md`.
- File a `Q-n` blocker describing the failure and attempts.
- Surface to the user and STOP. Do not advance to the next TASK.

### 7. Regression-sensitive re-run

After direct tests pass, run:
- All TCs that trace to any `RISK-n` in this TASK's `risk_refs`.
- The pre-existing test suite for modules touched (the module's own
  unit-test target).

New failures count as attempts (see §6).

### 8. Finalise the TASK

- Mark TASK `status: done` with ISO-8601 timestamp in `plan.md`.
- Append closing entry to `impl-log.md`: final files changed, TC statuses
  moved red→green, any spec deviations (pointer to `Q-n` if any).
- Update `test-matrix.csv` — flip `status` from `red` to `green` for the
  TCs this TASK turned green.
- Update `traceability.md` `Commits` column with placeholder `pending` (the
  user commits; `lint` will later cross-reference SHAs if supplied).

### 9. Knowledge-base / design-doc sync

If the change affects behaviour described in the host knowledge base or
formal design docs, update the relevant page(s) per the host's own
conventions. See **`rules-detail.md` §knowledge-base** for the full rule.

---

## Finalisation (after all TASKs done or blocked)

### A. Full test run across required targets

Run the test command for every build target in
`status.json.toolchain.targets` (or the default if single-target). Capture:
- Per-target build status.
- Per-target test results (pass / fail counts + failing test names).
- TC-level pass/fail mapped from the matrix.

### B. Write `03-impl/verify-report.md`

Use `../templates/verify-report.md`. Include:
- Summary table: per-target build ok, tests passing / total.
- AC coverage: for every AC, the corresponding TCs and final status.
- REQ/NFR coverage: every REQ/NFR and the AC / TC chain proving it.
- Open `Q-n` (advisory, resolved, blocked).
- Open `RISK-n` still valid (blocked mitigations).
- List of production files modified, grouped by module.
- List of new files added.
- Changelog line for the host repo (draft; user polishes before committing).

### C. Update `status.json`

Per **`references/status-schema.md`** §3:
- `stages.implement.done` = `true` iff `verify-report.md` shows 100 % of
  automated TCs green and no pre-existing test regressed.
- Set `stage: implement-done` on success. Otherwise leave at `tests-ready`
  and set `stages.implement.gates.green_state_confirmed: false`.

### D. Hand-off to the user

Do NOT commit. Print:

- "`implement` complete. Working tree is dirty; please review the diff and
  commit."
- Summary of changes (files, tests now green, ACs satisfied).
- Any open `Q-n` or `RISK-n` to look at before committing.
- Suggested commit message template (but do not execute `git commit`):

  ```
  <feature_key>: <one-line summary>

  Implements: REQ-1, REQ-2, NFR-1
  LLD: .specs/<FEATURE_KEY>/01-design/lld.md
  Tests: TC-1..TC-n (all green on <targets>)
  Closes: <FEATURE_KEY>
  ```

- Suggest running `/spec-lint` to confirm traceability before opening the MR.

## Self-validation

Run the protocol in **`references/self-validation.md`** with:

1. "For AC-3, which TCs are now green, and which TASKs made them green?" →
   `test-matrix.csv` + `traceability.md` + `impl-log.md`.
2. "List all spec deviations that occurred during implementation." →
   `impl-log.md` §Scope deviations + new `Q-n`s.
3. "Which pre-existing tests (if any) were affected, and how?" →
   `verify-report.md` regression section.

## Gates (must pass to advance `stage` to `implement-done`)

- Every automated TC in `test-matrix.csv` is `green`.
- Every TASK in `plan.md` is `done` (or explicitly `skipped` with
  justification in `impl-log.md`).
- No pre-existing test regressed.
- `verify-report.md` exists and covers every AC and REQ/NFR.
- No `Q-n` of severity `blocker` is unresolved.
- Working tree is clean of stray/accidentally-added files (obey ignore lists).
