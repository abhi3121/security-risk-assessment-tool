---
feature_key: hello-feature
stage: init
artifact: clarifications
last_updated: 2026-05-06
confidence: high
generator: skill-spec-driven-dev@0.1.0
---

# Clarifications — hello-feature

## Q-1 (advisory)
- Raised by: design
- Date: 2026-05-06
- Source: 01-design/lld.md#open-questions
- Question: Should names longer than 1024 chars be rejected, truncated, or accepted silently?
- Impact: Low — perf/correctness both fine in practice; user experience mildly affected.
- Proposed assumption: Accept silently (matches PM non-goal "any UTF-8 string is acceptable").
- Resolution: deferred — revisit if a real user complains.
