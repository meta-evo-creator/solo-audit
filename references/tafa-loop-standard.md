# TAFA Closed Loop · Standard Operating Procedure

> 2026-07-23, first complete closed-loop validation. From paper to SOP.

## Closed-loop flow

```
Audit layer starts  →  21:00 Pro model runs the audit automatically
  violation found   →  P0 (human approval) / P1 (automatic proposal) / OBS (record only)
  ⏵ routing         →  each violation annotated with the target capability-layer component + suggested fix
Capability layer responds → analyze root cause → propose fix plan → impact scope → rollback plan
Authority-layer approval  →  Human in the Loop: "do it" or "no"
Capability-layer execution → fix payload / code / config
🛡️ Guardrail consolidation → attribution (where the error was) → localization (where to plug it) → consolidation (write into the rules) → verification (does it actually work?)
Audit-layer verification  →  the next audit checks whether the previous period's violations recur

Timeout rules: P1 >24h → escalate to P0 | execution >12h → remind | consolidation >48h → OBS
```

## Responsibilities of each layer (no crossing boundaries)

| Layer | Does | Does not do |
|:---|:------|:--------|
| Audit layer (Pro · read-only) | Audit · tier · route · verify | Modify code · propose plans · execute beyond authority |
| Capability layer (Flash · read-write) | Analyze · propose · execute · consolidate | Approve · decide on its own · skip the authority layer |
| Authority layer (the owner) | Approve "do it / no" | Direct · audit · fix by itself |

## Inherent limitation

Audit layer = capability layer (same body, the answering agent). Mitigation: Pro + read-only + evidence guardrail.
This is already the best available option. If an independent audit-layer system becomes available in the future, it can be upgraded.

## First closed-loop record

SP-030 · 2026-07-23 22:48-23:10 · 15 minutes
3 P1 items · all closed · see memory/milestones/2026-07-23-tafa-first-loop.md for details
