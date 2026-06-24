---
feature_key: FEATURE-KEY
stage: design
artifact: plan
last_updated: YYYY-MM-DD
confidence: medium
generator: skill-spec-driven-dev@0.1.0
---

# Implementation Plan — <feature title>

> Ordered `TASK-n` list grouped by phase. Every task is small enough that a
> single PR could deliver it (rule of thumb: ≤ 1 day of work). `status` is
> one of: `todo`, `in-progress`, `done`, `blocked`, `skipped`.

## Phase 1 — Scaffold

```yaml
- id: TASK-1
  title: "Add skeleton files and build-system wiring for <feature>"
  description: |
    Create empty source files, headers, and hook them into the build. No
    behaviour yet.
  status: todo
  depends_on: []
  lld_refs:  [LLD-1]
  ac_refs:   []
  risk_refs: []
  files:
    - firmware_vob/SOURCE/LUNA2/CRY_MOD/cry_feature.c
    - firmware_vob/SOURCE/LUNA2/CRY_MOD/cry_feature.h
    - firmware_vob/SOURCE/LUNA2/CRY_MOD/CMakeLists.txt
  validation:
    build: "<detected build command>"
    tests: []                 # filled in by /spec-tests
  est_effort: S
```

## Phase 2 — Core

```yaml
- id: TASK-2
  title: "Implement happy-path keygen entry point"
  description: |
    Add NewFunction() that drives the underlying primitive. Covers AC-1.
  status: todo
  depends_on: [TASK-1]
  lld_refs:  [LLD-1, LLD-3]
  ac_refs:   [AC-1]
  risk_refs: [RISK-1]
  files:
    - firmware_vob/SOURCE/LUNA2/CRY_MOD/cry_feature.c
  validation:
    build: "<detected build command>"
    tests: []                 # filled in by /spec-tests
  est_effort: M
```

## Phase 3 — Integration

```yaml
- id: TASK-3
  title: "Wire ICD command 0xNNNN to the new entry point"
  description: |
    Register the command and its argument decoder in COM_MOD's dispatch table.
  status: todo
  depends_on: [TASK-2]
  lld_refs:  [LLD-2]
  ac_refs:   [AC-2]
  risk_refs: []
  files:
    - firmware_vob/SOURCE/LUNA2/COM_MOD/com_dispatch.c
  validation:
    build: "<detected build command>"
    tests: []
  est_effort: S
```

## Phase 4 — Hardening

```yaml
- id: TASK-4
  title: "Add negative-path handling and error taxonomy"
  description: |
    Validate inputs, return structured errors, add log lines per the LLD.
  status: todo
  depends_on: [TASK-2]
  lld_refs:  [LLD-1]
  ac_refs:   [AC-3]
  risk_refs: []
  files:
    - firmware_vob/SOURCE/LUNA2/CRY_MOD/cry_feature.c
  validation:
    build: "<detected build command>"
    tests: []
  est_effort: S
```

<!-- Keep each task small. Split any TASK with est_effort=L and no justification. -->
