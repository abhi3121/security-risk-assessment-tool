---
feature_key: FEATURE-KEY
stage: init
artifact: readme
last_updated: YYYY-MM-DD
confidence: medium
generator: skill-spec-driven-dev@0.1.0
---

# <FEATURE_KEY> — <one-line title>

Workspace for the skill-spec-driven-dev skill. This directory is the single source of
truth for this feature's specs, tests, and implementation record.

## Quick links

- [Inputs manifest](00-inputs/inputs.yaml)
- [Context manifest](00-inputs/context-manifest.md)
- [Low-level design](01-design/lld.md)
- [Acceptance criteria](01-design/acceptance.md)
- [Implementation plan](01-design/plan.md)
- [Risks](01-design/risks.md)
- [Test spec](02-tests/tests.md) · [Test matrix (CSV)](02-tests/test-matrix.csv)
- [Implementation log](03-impl/impl-log.md) · [Verify report](03-impl/verify-report.md)
- [Traceability matrix](traceability.md)
- [Clarifications](clarifications.md)
- [Machine state (status.json)](status.json)

## Summary

<2-3 sentences describing the feature.>

## Current state

- **Stage:** inputs-ready | design-ready | tests-ready | implement-done
- **Open clarifications (blockers):** 0
- **Open clarifications (advisory):** 0
- **Open risks:** 0

## How to continue

- To progress to the next stage, run the matching capability from the
  `skill-spec-driven-dev` skill (`init` → `design` → `tests` → `implement`), or run
  `pipeline` to chain them with human checkpoints.
- To check traceability health at any time, run `lint`.
- Working tree changes produced by `implement` are NEVER auto-committed.
  Review the diff before committing.
