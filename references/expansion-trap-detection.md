# Rule Expansion Trap Detection Pattern

> Source: AUDIT-20260704 audit findings
> Applies to: solo-audit daily audit §3.2 VERIFY

## Detection signals

The following patterns indicate that a "rule expansion trap" may be underway:

1. **The same violation recurs across weeks** — the same type of violation appears ≥3 times within 15 days with no improvement
2. **Design-layer patching does not change execution metrics** — after a new rule/guardrail is added, the related metrics do not change (e.g. session expansion rate, citation accuracy)
3. **Rule-addition frequency > execution-verification frequency** — compare the number of `patch` calls in a session with the number of `rg` verification calls

## Detection method

```
1. Compare the overlap between this audit's violations and those from 30 days ago
2. If overlap > 30% → mark [EXPANSION_TRAP_CHECK]
3. Check whether the overlapping items have a corresponding new rule/guardrail
4. If a new rule exists but there is no improvement → escalate to [EXPANSION_TRAP_CONFIRMED]
```

## Response

- [EXPANSION_TRAP_CONFIRMED] → change the proposal type to "path removal"; the goal is not to add rules but to delete the path that produces the problem
- The proposal must include: "The existing rule X already covers this problem; the root cause is in the execution layer, not the design layer"
