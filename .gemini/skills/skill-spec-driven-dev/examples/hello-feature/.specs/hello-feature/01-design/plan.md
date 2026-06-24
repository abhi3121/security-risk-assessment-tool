---
feature_key: hello-feature
stage: design
artifact: plan
last_updated: 2026-05-06
confidence: medium
generator: skill-spec-driven-dev@0.1.0
---

# Implementation Plan — hello-feature

## Phase 1 — Scaffold

```yaml
- id: TASK-1
  title: "Create src/cli/hello.ts skeleton and wire into dispatcher"
  description: |
    Create an empty hello.ts exporting stubbed hello() and handleHello(),
    and register the "hello" route in src/cli/index.ts. No behaviour yet.
  status: done
  depends_on: []
  lld_refs:  [LLD-1, LLD-2]
  ac_refs:   []
  risk_refs: []
  files:
    - src/cli/hello.ts
    - src/cli/index.ts
  validation:
    build: "npm run build"
    tests: [TC-1]     # build must still succeed; TC-1 remains red
  est_effort: S
```

## Phase 2 — Core

```yaml
- id: TASK-2
  title: "Implement hello(name) pure function"
  description: |
    Return `Hello, ${name}!`. Turns TC-1, TC-3, TC-4, TC-6 green.
  status: done
  depends_on: [TASK-1]
  lld_refs:  [LLD-1]
  ac_refs:   [AC-1, AC-3, AC-4, AC-6]
  risk_refs: []
  files:
    - src/cli/hello.ts
  validation:
    build: "npm run build"
    tests: [TC-1, TC-3, TC-4, TC-6]
  est_effort: S

- id: TASK-3
  title: "Implement handleHello(argv) including usage error"
  description: |
    stdout/stderr/exit-code behaviour per LLD-2. Turns TC-2 green.
  status: done
  depends_on: [TASK-2]
  lld_refs:  [LLD-2]
  ac_refs:   [AC-2]
  risk_refs: []
  files:
    - src/cli/hello.ts
  validation:
    build: "npm run build"
    tests: [TC-2]
  est_effort: S
```

## Phase 3 — Hardening

```yaml
- id: TASK-4
  title: "Add performance benchmark for NFR-1"
  description: |
    Vitest bench: hello() over names of length 1, 64, 1024, N=1000.
    Turns TC-5 green.
  status: done
  depends_on: [TASK-2]
  lld_refs:  [LLD-1]
  ac_refs:   [AC-5]
  risk_refs: []
  files:
    - tests/hello.bench.ts
  validation:
    build: "npm run build"
    tests: [TC-5]
  est_effort: S
```
