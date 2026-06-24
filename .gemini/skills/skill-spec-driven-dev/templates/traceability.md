---
feature_key: FEATURE-KEY
stage: init
artifact: traceability
last_updated: YYYY-MM-DD
confidence: high
generator: skill-spec-driven-dev@0.1.0
---

# Traceability Matrix — <feature title>

> Append-only. Rows are added as new relationships are discovered; never
> silently modified. Empty cell = `—`, never blank. `status` is one of:
> `needs-design` · `designed` · `tested` · `implemented` · `superseded` ·
> `retired` · `pending`.

| REQ / NFR | AC    | LLD     | TASK    | TC            | Commits | Status       |
|-----------|-------|---------|---------|---------------|---------|--------------|
| REQ-1     | —     | —       | —       | —             | —       | needs-design |
| NFR-1     | —     | —       | —       | —             | —       | needs-design |

<!-- Example rows after later stages:

| REQ-1     | AC-1  | LLD-3   | TASK-2  | TC-1          | pending | designed     |
| REQ-1     | AC-1  | LLD-3   | TASK-2  | TC-1          | <sha>   | implemented  |

-->
