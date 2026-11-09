# Expansion Trap Guardrail — Enforced Standard

> Cornerstone of the five principles. Not a reminder — a gate.

## Trigger condition

Mandatory trigger before modifying these files: SOUL.md / AGENTS.md / MEMORY.md / skill SKILL.md / agent .md

## Execution steps

Before modifying, you must first output:

```
I am about to make the following change in [file]:
  Add: [N] lines — [description of the specific content]
  Delete: [M] lines — [description of the old content being replaced]
  Net change: [N-M] lines
  Reason: [why this cannot be solved by removing a path?]
```

## Gate rules

| Net change | Result |
|:--:|:-----|
| ≤0 | ✅ compression / refactor |
| >0 with corresponding deletion | ✅ replacement |
| >0 without corresponding deletion | 🔴 expansion — requires approval |

## Case (7/21-22)

SOUL.md +42 lines → -42 lines | solo-audit +70 lines → -70 lines | DI refs 31 → 16 files | Cron payload: MEV matrix removed
