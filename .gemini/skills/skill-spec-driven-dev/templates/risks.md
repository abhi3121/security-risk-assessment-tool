---
feature_key: FEATURE-KEY
stage: design
artifact: risks
last_updated: YYYY-MM-DD
confidence: medium
generator: skill-spec-driven-dev@0.1.0
---

# Risk Register — <feature title>

> One `RISK-n` per significant risk. Every blocker `Q-n` should have a
> corresponding `RISK-n` OR be resolved before design exits.

## RISK-1
- **Description:** <one line>
- **Likelihood:** low | medium | high
- **Impact:** low | medium | high
- **Mitigation:** <specific, actionable>
- **Owner:** <name or role>
- **Related:** LLD-?, TASK-?, Q-?
- **Status:** open | mitigated | accepted | retired

## RISK-2
- **Description:** Dependency `liboqs` pin may lag upstream security advisories.
- **Likelihood:** medium
- **Impact:** high
- **Mitigation:** Subscribe to CVE feed; quarterly refresh task TASK-N.
- **Owner:** crypto team
- **Related:** LLD-5, Q-3
- **Status:** open

<!-- Categories to consider:
     - Unknowns / underspecified areas (link each to a Q-n)
     - External dependencies / third-party instability
     - Performance / scale risks
     - Security / crypto / data-handling risks
     - Migration / rollback risks
     - Certification / compliance risks
-->
