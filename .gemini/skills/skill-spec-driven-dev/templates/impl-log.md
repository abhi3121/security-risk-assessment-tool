---
feature_key: FEATURE-KEY
stage: implement
artifact: impl-log
last_updated: YYYY-MM-DD
confidence: high
generator: skill-spec-driven-dev@0.1.0
---

# Implementation Log — <feature title>

> Append-only journal kept by the `implement` capability. One section per
> `TASK-n` as it executes. New `Q-n` / scope deviations recorded here AND in
> `clarifications.md` when relevant.

---

## TASK-1 — started YYYY-MM-DDTHH:MM:SSZ
- **Goal:** <one sentence>
- **Must turn green:** TC-?, TC-?
- **Intended file changes:** <files from plan.md>
- **Relevant LLD sections:** LLD-?, LLD-?
- **Notes / assumptions:**

### Build & test attempts
1. Attempt 1 (YYYY-MM-DDTHH:MM:SSZ):
   - Build: ok / fail (<error summary>)
   - Tests: TC-1 fail → pass, TC-2 fail → fail
   - Action taken: …

### Test fixes (if any)
- TC-2: fixed bad fixture (off-by-one in expected size). Not a production bug.

### Scope deviations (if any)
- Added `cry_feature_internal.h` outside declared files — approved by user on YYYY-MM-DD.

### Outcome — finished YYYY-MM-DDTHH:MM:SSZ
- **Status:** done | blocked | skipped
- **Files changed (final):**
  - `path/to/file.ext` (new | modified)
- **TCs turned green:** TC-1, TC-2
- **New Q-n filed:** Q-?  (if any)

---

<!-- Repeat the above block for each TASK-n -->
