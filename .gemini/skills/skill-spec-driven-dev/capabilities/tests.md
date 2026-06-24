# Capability: tests — TDD test specification and RED-state skeletons

Produce a comprehensive test specification (unit + integration + e2e as
appropriate) and generate failing test skeletons in the host repo's test
framework so the suite is RED before any production code is written.

## Preconditions

- `status.json.stage` ∈ { `design-ready`, `tests-ready` }  (re-runnable)
- `01-design/{lld.md, acceptance.md, plan.md, risks.md}` exist
- No `Q-n` of severity `blocker` unresolved

## Read order

1. `01-design/acceptance.md` — primary source of truth for tests
2. `01-design/lld.md` (especially Interfaces, Data model, Edge cases,
   Constraints)
3. `01-design/risks.md` — to identify regression-sensitive areas
4. `01-design/plan.md` — to know which tasks need which tests as their
   `validation.tests`
5. `00-inputs/inputs.yaml` — to rediscover NFRs and measurable thresholds
6. `status.json.toolchain.test_framework` — to pick the correct skeleton
7. If a knowledge base is present: pages about existing test conventions /
   patterns / fixtures (see **`rules-detail.md` §knowledge-base**)

## Outputs

- `02-tests/tests.md` — human-readable test specification
- `02-tests/test-matrix.csv` — machine-readable TC → AC/REQ/LLD map
- Failing test skeletons under the host repo
- Updates: `01-design/plan.md` (fill `validation.tests` with real TC ids),
  `traceability.md`, `status.json`, `clarifications.md`

## Steps

### 1. Confirm / refine test framework

If `status.json.toolchain.test_framework` was previously detected, confirm
with the user (or proceed silently if a pipeline gate is set). Otherwise
prompt:

> "Which test framework should I target for the new tests?
> Candidates detected: <list>. Or specify: <user-input>."

Record any update in `status.json`. Look up the conventional file location
and naming for the chosen framework in **`references/frameworks.md`** §3.
If the host repo has its own convention, honour it instead and ask before
introducing a new one.

### 2. Enumerate test cases → `02-tests/tests.md`

Use `../templates/tests.md`. For every `AC-n`, generate ≥1 `TC-n`. Cover:

- **Happy path** — expected success.
- **Edge cases** — from `lld.md` §Edge cases; each edge case that maps to an
  AC has a TC. For edge categories not yet considered, walk
  **`references/risk-categories.md`** §2 and pick those that apply.
- **Error paths** — invalid inputs, out-of-range, nulls, races, resource
  exhaustion, timeouts.
- **Regression-sensitive areas** from `risks.md` — add TCs that would catch
  the risk.
- **NFR validation** — performance budgets, security invariants, compatibility.
- **UI / UX coverage** — when `inputs.yaml.ui_inputs.applicable: true`,
  every screen / route in LLD `LLD-UI-1` MUST have ≥1 TC. Include at least
  one each of: component-render TC (RTL / Vue Test Utils / Angular TestBed),
  interaction TC, accessibility TC (axe / `@axe-core/*` smoke; assert no
  critical violations; map to LLD-UI-4), visual / responsive TC (Playwright
  screenshot or Storybook visual snapshot at LLD-UI-5 breakpoints — mark
  `level: e2e` or `manual` if no infra exists). Fall back to `manual` only
  if no automated path exists; file an advisory `Q-n`.

Each TC entry:

```markdown
## TC-1  [unit|integration|e2e|performance|security|manual]  (AC-1)
- **Traces:** REQ-1, NFR-1, LLD-3
- **Framework target:** gtest+ctest
- **Location:** firmware_vob/SOURCE/LUNA2/CRY_MOD/tests/ut_mlkem.cc::TC_1_keygen_happy
- **Preconditions:** <setup>
- **Inputs / fixtures:** <data>
- **Steps:** 1. ... 2. ... 3. ...
- **Expected:** <observable outcome>
- **Targets:** amd64-release, arm64-release
- **Notes:** <edge-case rationale, flakiness risk, etc.>
```

Prefer the smallest effective scope:
- `unit` for pure functions, data structures, simple stateful objects.
- `integration` when crossing a module / library / process boundary matters.
- `e2e` only when the contract is user-observable end-to-end.
- Classify NFR checks explicitly: `performance`, `security`.
- If an AC cannot be automated, use kind `manual` and describe the manual
  verification checklist.

### 3. Write `02-tests/test-matrix.csv`

Stable header (one row per TC):

```
tc_id,level,ac_refs,req_refs,lld_refs,framework,location,targets,status
TC-1,unit,AC-1,REQ-1,LLD-3,gtest,firmware_vob/.../ut_mlkem.cc::TC_1_keygen_happy,amd64-release;arm64-release,red
```

`status` enum: `red` | `green` | `skipped` | `manual` | `blocked`.
Initially `red` for automated tests; `manual` for manual ones.

### 4. Generate failing test skeletons in-tree

For each non-manual TC, write a skeleton at the declared `Location` that
**fails by default**. Use the framework-specific pattern from
**`references/frameworks.md`** §4 — copy only the row matching your detected
framework. Conventions for every skeleton:

- Put the `TC-n` id in the test name verbatim, so `grep TC-` works.
- If helpers/mocks don't exist yet, add minimal stubs in `fixtures/` /
  `mocks/` — stubs may `return FAIL;` or equivalent. **Never add production
  behaviour in this stage.**
- Wire the new test files into the build system if required by your
  toolchain. See **`references/frameworks.md`** §5 for build-system wiring
  per framework.

### 5. Build & run — confirm RED state

Run the build then the test command. Capture:
- Build must succeed (compilation + linking).
- New TCs must fail; pre-existing tests must still pass. If a pre-existing
  test regressed, STOP, file a blocker `Q-n`, do not advance.

Record under `status.json.stages.tests.gates`:
- `red_state_confirmed: true` iff build is green AND new tests are all
  failing as expected AND no pre-existing test newly failed.
- Otherwise `false` with `red_state_notes` describing the issue.

### 6. Back-fill TASK validation in `plan.md`

For each `TASK-n` in `01-design/plan.md`, set `validation.tests` to the list
of `TC-n` ids that task must turn green. Every AC must have ≥1 matching
TASK whose `validation.tests` contains the TC covering that AC.

### 7. Update `traceability.md`

Append rows so every `TC-n` is represented. Update existing rows that now
have known TCs. Keep prior rows intact (append-only).

### 8. Coverage gates

Block the stage if any hold:
- An AC has zero automated TC and is not explicitly classified `manual`.
- A REQ/NFR has no transitive TC via its ACs.
- A TASK has empty `validation.tests`.
- Build failed or a pre-existing test regressed.

Report which gate failed and what to fix.

### 9. Self-validation

Run the protocol in **`references/self-validation.md`** with:

1. "For AC-3, which TCs cover it and where do they live in the tree?" →
   `tests.md` + `test-matrix.csv`.
2. "List every TC of level=performance and the NFR it traces to." →
   `tests.md`.
3. "Did any pre-existing test break when the new skeletons were added?" →
   last test-run log in `status.json.stages.tests.gates.red_state_notes`.

### 10. Report to the user

Print:
- Counts: total TCs by level, ACs covered / total, NFRs covered / total.
- Build + test result summary (RED confirmed? which tests failed and why).
- List of new files added to the tree.
- Next step: run `implement`.

## Gates (must pass to advance `stage` to `tests-ready`)

- Every AC has ≥1 TC (automated OR explicitly `manual`).
- Every REQ/NFR is transitively covered by a TC.
- Every TASK has a non-empty `validation.tests` list.
- Build succeeds with the new test files.
- New TCs fail as expected; no pre-existing test regressed.
- `test-matrix.csv` is well-formed (header row + one row per TC).
