# TAFA Closed Loop: Complete Requirement from Audit Report to Proposal

> 2026-07-24 correction by the owner. Core of the v8.3 → v8.4 upgrade.

## Problem

Audit v8.3's report listed violations (P0/P1/OBS) and annotated "⏵ route to capability layer [xxx]", but did not generate a concrete executable proposal.
The owner: "Don't you understand what the TAFA closed loop means? Beyond giving an audit report, the daily audit also has to route to each capability layer and give the corresponding proposal."

## Correction: v8.4 audit output format

Every violation must be accompanied by:

| Level | Proposal must contain | Example |
|:----:|:----------|:-----|
| 🔴 P0 | File path · specific change · impact scope | `config.yaml:2 — default: the Pro-tier model → the fast-tier model` |
| ⚠️ P1 | File path · change suggestion · timeout | `PLUGIN-REGISTRY.md — add solo-fleet entry · >24h → escalate to P0` |
| 🔵 OBS | Record only · deterioration escalation condition | `Deterioration condition: injection budget >35KB → escalate to P1` |

## Proposal template

```
⏵ **Routed to**: capability layer [specific executing component]
📋 **Proposal content**: [specific operation steps · file path · expected effect]
⏱️ **Recommended execution**: [immediately / within 24h / before the next audit]
```

## Verification

2026-07-24 21:40, audit v8.4 ran with proposals attached for the first time. P001/P002/P003/P004 all contained specific paths + operation steps + time limits.
