# Capability: pipeline — End-to-end orchestrator

Run `init → design → tests → implement` back-to-back, pausing for a human
checkpoint between each stage. Pipeline adds no behaviour of its own; it
delegates to the four stage capability files and enforces the gate checks
declared in each.

## Preconditions

- None (pipeline handles both fresh and resumed features).

## Inputs (prompted at start)

1. **Mode** — one of:
   - `interactive` (default): pause between each stage and ask
     "proceed to <next stage>? (y/N)".
   - `yes-to-all`: run end-to-end without pausing, but STOP on any gate
     failure or blocker clarification.
   - `dry-run`: report which stages would run given current state without
     executing.
2. **Starting stage** — auto-detect from `status.json.stage` if the workspace
   exists; otherwise start at `init`.
3. **Stop-after** — optional stage to stop at (e.g. "stop after `tests` so I
   can review skeletons before implementation").

## Steps

### 1. Load state

If a workspace path is supplied or detectable, read `status.json`. Otherwise
treat this as a fresh run and proceed to `init`.

### 2. Determine stage plan

Based on current `stage` and `stop-after`:

| Current stage       | Default plan                                      |
|---------------------|---------------------------------------------------|
| (no workspace)      | init → design → tests → implement                  |
| `inputs-blocked`    | STOP — instruct user to resolve `Q-n` blockers    |
| `inputs-ready`      | design → tests → implement                         |
| `design-ready`      | tests → implement                                  |
| `tests-ready`       | implement                                          |
| `implement-done`    | optional lint only                                 |

In dry-run mode, print this plan and exit.

### 3. Run stages

For each stage in the plan, in order:

a. Print banner: `=== stage: <name> ===`.
b. Load only `capabilities/<stage>.md` (and any reference files its steps
   explicitly point at). Execute per its own rules. Unload the capability
   file from working context after the stage completes.
c. After the stage finishes, read the updated `status.json`:
   - If the stage's gates did NOT all pass, STOP and surface the failing
     gates. Do not advance.
   - If any `Q-n` of severity `blocker` was raised, STOP and surface it.
d. In `interactive` mode, ask:
   > "Stage `<name>` completed successfully. Proceed to `<next>`? (y/N)"

   On `n` or no response, stop gracefully — workspace is left in a
   resumable state.
e. In `yes-to-all` mode, skip the prompt and continue.

### 4. After final stage

- Run `lint` as a final verification pass (does not modify artifacts).
  Print the summary.
- Print a consolidated report:
  - Feature key and workspace path
  - Requirements → AC → TC → TASK coverage counts
  - Open `Q-n` (blocker / advisory)
  - Open `RISK-n`
  - Suggested next action (e.g. "review diff and commit", "resolve 2
    advisory Q-n", etc.)

## Resuming a pipeline

If `pipeline` is re-invoked on an existing workspace:
- Re-read `status.json`.
- Offer to continue from the current stage, or re-run a prior stage (which
  resets subsequent stages' `done` flag).

## Notes

- Pipeline is a convenience layer. For non-trivial features the recommended
  path is running each capability manually with human review between stages.
- Pipeline does not replace human review. Even in `yes-to-all` mode, the
  user MUST review the diff before committing (`implement` never auto-commits).
