---
feature_key: FEATURE-KEY
stage: design
artifact: acceptance
last_updated: YYYY-MM-DD
confidence: medium
generator: skill-spec-driven-dev@0.1.0
---

# Acceptance Criteria — <feature title>

> One `AC-n` per testable criterion. Every `REQ-*` and `NFR-*` MUST be covered
> by at least one AC. Gherkin form preferred; `kind` is one of
> `automatable` (default), `manual`, `performance`, `security`.

## AC-1  (REQ-1)
- **kind:** automatable
- **Given** <precondition>
- **When**  <action>
- **Then**  <observable outcome>
- **Notes:** measurement technique, tolerances, data sets.

## AC-2  (REQ-2, NFR-1)
- **kind:** performance
- **Given** …
- **When**  …
- **Then**  p95 latency ≤ 50 ms on amd64-release (measured over N=1000 runs).
- **Notes:** benchmark fixture under `…`.

## AC-3  (REQ-3)  — negative path
- **kind:** automatable
- **Given** invalid input X
- **When**  the caller invokes Y
- **Then**  the function returns `ERROR_INVALID_ARG` and no state changes.

<!-- Add as many AC-n as needed. Delete this placeholder set on first real use. -->
