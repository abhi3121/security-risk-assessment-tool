---
feature_key: hello-feature
stage: tests
artifact: tests
last_updated: 2026-05-06
confidence: medium
generator: skill-spec-driven-dev@0.1.0
---

# Test Specification — hello-feature

## TC-1  [unit]  (AC-1)
- **Traces:** REQ-1, LLD-1
- **Framework target:** vitest
- **Location:** `tests/hello.test.ts::TC_1_hello_happy`
- **Steps:** call `hello("Alice")`
- **Expected:** `"Hello, Alice!"`

## TC-2  [integration]  (AC-2)
- **Traces:** REQ-2, LLD-2
- **Framework target:** vitest
- **Location:** `tests/hello.test.ts::TC_2_usage_error`
- **Steps:** call `handleHello([])`; capture stderr + return code
- **Expected:** stderr exactly `"usage: hello <name>\n"`; return value 2

## TC-3  [unit]  (AC-3)
- **Traces:** REQ-1
- **Location:** `tests/hello.test.ts::TC_3_whitespace_name`
- **Steps:** call `hello("   ")`
- **Expected:** `"Hello,    !"`

## TC-4  [unit]  (AC-4)
- **Traces:** REQ-1
- **Location:** `tests/hello.test.ts::TC_4_long_name`
- **Steps:** call `hello("a".repeat(1024))`
- **Expected:** returns `` `Hello, ${"a".repeat(1024)}!` ``

## TC-5  [performance]  (AC-5, NFR-1)
- **Traces:** NFR-1
- **Framework target:** vitest bench
- **Location:** `tests/hello.bench.ts::TC_5_perf`
- **Steps:** run 1000 iterations at lengths 1, 64, 1024
- **Expected:** p95 < 10 ms per call on node20

## TC-6  [unit]  (AC-6)
- **Traces:** REQ-1
- **Location:** `tests/hello.test.ts::TC_6_unicode_name`
- **Steps:** call `hello("🙂")`
- **Expected:** `"Hello, 🙂!"`
