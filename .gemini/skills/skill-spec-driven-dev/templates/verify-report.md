---
feature_key: FEATURE-KEY
stage: implement
artifact: verify-report
last_updated: YYYY-MM-DD
confidence: high
generator: skill-spec-driven-dev@0.1.0
---

# Verification Report — <feature title>

> Produced at the end of the `implement` capability. Source of truth for
> "did this feature actually ship?" Review this before committing.

## Build summary

| Target          | Build | Notes |
|-----------------|-------|-------|
| amd64-release   | ok    | —     |
| arm64-release   | ok    | —     |

## Test summary

| Target          | Tests run | Passed | Failed | Skipped |
|-----------------|-----------|--------|--------|---------|
| amd64-release   | 42        | 42     | 0      | 0       |
| arm64-release   | 42        | 42     | 0      | 0       |

## Acceptance coverage

| AC   | TCs                    | All green? | Notes |
|------|------------------------|------------|-------|
| AC-1 | TC-1, TC-4             | ✅         | —     |
| AC-2 | TC-2, TC-3 (perf)      | ✅         | p95 = 42 ms |
| AC-3 | TC-5                   | ✅         | —     |

## Requirement coverage

| REQ / NFR | Via ACs       | Via TCs        | Final status |
|-----------|---------------|----------------|--------------|
| REQ-1     | AC-1, AC-3    | TC-1, TC-4, TC-5 | implemented |
| NFR-1     | AC-2          | TC-3            | implemented  |

## Regression check

- Pre-existing tests: all green.
- Any tests newly marked `skipped`? Reason:

## Open items

### Unresolved `Q-n`
- Q-?  (advisory — documented assumption)

### Active `RISK-n`
- RISK-?  (mitigation in place: …)

## Files changed

### Production
- `path/to/file.ext` (modified)
- `path/to/newfile.ext` (new)

### Tests
- `path/to/test.ext` (new)

### Docs / knowledge base
- `.wiki/modules/…` (updated)
- `firmware_vob/SOURCE/LUNA2/<MOD>/docs/<feature>.md` (new)

## Suggested commit message (draft)

```
<feature_key>: <one-line summary>

Implements: REQ-1, REQ-2, NFR-1
LLD:        .specs/<FEATURE_KEY>/01-design/lld.md
Tests:      TC-1..TC-n (all green on amd64-release, arm64-release)
Closes:     <FEATURE_KEY>
```

## Next steps for the human

1. Review the diff.
2. Run `spec-lint` for a final traceability check.
3. Resolve any remaining advisory `Q-n` or defer with explicit comment.
4. Commit and open MR.
