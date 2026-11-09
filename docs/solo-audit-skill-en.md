---
name: solo-audit-en
version: 6.13.0
description: "TAFA-native self-audit: iron rules, operating principles, reasoning loop, structural guardrail verification, trace completeness. 4-agent pipeline. Finds and routes only — never fixes."
---

# SOLO Audit (English edition)

> Public, desensitised edition of the audit skill. Client material, organisational identity,
> personnel, paths, credentials and model identifiers are excluded.

## When to use

- A scheduled audit run of the fleet.
- The owner asks "what is wrong right now", "is the guardrail actually working", "did anything break".
- Before a structural change ships (new skill, new interceptor, new scheduled job).
- Not for: building features, fixing defects, drafting content, or grading a single document.

## Hard rules

1. **Find and route, never fix.** Fixes travel the capability loop: proposal → approval → execution.
2. **No evidence, no finding.** Inspect it yourself; an inherited claim is re-verified before it becomes a finding.
3. **Severity is not a mood.** P0 = privacy, authority, data loss, production impact, or a guardrail that does not fire. P1 = path, format, verification, threshold. OBS = trend, performance.
4. **Zero findings = zero output.** Silence is the correct result of a clean run.
5. **Never move the ruler.** Criteria belong to the authority layer. Changing a threshold so an alert disappears is itself a violation.

## Scope

| Area | What is checked |
|---|---|
| Iron rules | sourcing, authority, honesty, file care, injection resistance, no fabrication |
| Principles | inflation trap, parsimony, priority, entropy, guardrail reality |
| Reasoning loop | five stages actually traversed, or skipped and called analysis |
| Performance | observation only, never a verdict |
| Infrastructure | real end-to-end probes of remote capabilities; scheduled-job state; ageing proposals |
| Trace | case-type artifacts present, appended not overwritten, no sensitive egress |

## Pipeline

```
COLLECT (audit) → VERIFY (audit) → ANALYZE (capability) → REPORT (audit)
```

- **COLLECT** — batch evidence collection into one structured file. Probe remote capabilities for real: a handshake proves a port, a query proves the tool.
- **VERIFY** — drop false positives, confirm cross-day recurrence, re-check anything inherited from the previous ledger. Run probes with the runtime environment the job actually uses.
- **ANALYZE** — classify root cause, assign severity, file a proposal card. "To be analysed" is not a card.
- **REPORT** — one block per finding: `capability[action] → violates → authority[rule]`, plus evidence, level, route. Then the proposal queue, then the guardrail-hardening note.

## Evidence gates

- One failed probe is not a finding: retry once, then escalate. Transient failure is OBS.
- A previously reported item whose artifact changed after the failure is resolved — do not re-report.
- Count availability from detail lines, deduplicated by name; summary lines double-count.
- A finding whose three questions (what happened, which rule, did I see it) cannot all be answered is deleted.

## Guardrail verification

Exercise each structural interceptor with a call that must be blocked. Allowed, or exit code 0, on
such a call = **P0, guardrail ineffective**. Documentation proves nothing; interception proves
everything.

## Trace check

| Condition | Level |
|---|---|
| trace file missing on a case-type task | P1 |
| artifact overwritten instead of appended | P1 |
| sensitive material leaving the local boundary | P0 |

Ledger: one row per round `{date, tasks, missing, overwrites, egress, verdict}`. A ledger older than
30 days counts as not meeting standard and triggers a mandatory re-review.

## Delivery

| Outcome | Behaviour |
|---|---|
| zero violations | silent exit |
| violations | final message is the audit summary |
| P0 | flagged at the top, authority layer notified immediately |

## Known failure patterns

- **Criteria drift** — a label, selector, interpreter or path stops matching reality; the check becomes a permanent false alarm or a permanent false pass. Re-verify against source, then reverse-test the fix.
- **Quota misrouted as defect** — every variant of an error string must map to the right branch; mis-routing is worse than not detecting.
- **Interactive job with no session** — "started" with no logged-in user does nothing. Check user, timestamp, process together.
- **Collection mistaken for judgement** — any threshold must be evaluated after collection, never inferred from the collection method.
