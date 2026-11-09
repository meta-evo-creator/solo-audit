# TAFA Closed-Loop Completeness — 2026-07-24 Field Lesson

## Problem
After the audit cron v8.3 ran, it only output a violation list and did not route to the capability layer to generate executable proposals.
User correction: an audit report ≠ completion; after the report, routing proposals must be attached.

## Fix
v8.4 upgrade — every audit output must be accompanied by:
- ⏵ routing of each violation to a specific capability layer
- 📋 executable proposal (file path · operation steps · time limit)
- ⏱️ recommended execution time

## Complete closed loop
```
Audit-layer audit → violation found → ⏵ route to capability layer → capability-layer proposal → authority-layer approval
  → capability-layer execution → capability-layer guardrail → next audit-layer verification
```
Missing any step = the closed loop is broken.
