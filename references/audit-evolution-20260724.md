# SOLO Audit Methodology Evolution (2026-07-24)

## Three-level triage (P0/P1/OBS) final version

| Level | Approval | Timeout | Determination |
|:----:|:----:|:----:|:-----|
| 🔴P0 | Authority layer approves the plan | Never expires; no automatic escalation | Privacy, authority and accountability, data loss, production |
| ⚠️P1 | Authority layer approves the plan | No approval for >24h → automatically escalated to P0 | Path, format, verification, threshold |
| 🔵OBS | — | Deterioration → escalated to P1 | Performance, trend, non-urgent |

Both P0 and P1 require authority-layer approval before execution — the difference is the timeout behavior. P0 waits forever; P1 escalates automatically after too long to draw attention.

## Four steps of guardrail-style consolidation

Attribution (where the error was) → localization (where to plug it) → consolidation (write it into the rules) → trigger verification (does it actually work?)
Verification method: after each guardrail consolidation, run one retest against the fix point.

## DI hard block

If a user message contains the keywords "interview manual / interview outline / adjudication recommendation / clue" → the main session is forbidden to answer directly → it must delegate to the DI pipeline.
Self-check: "Is my answer a DI matter? → If so, stop."

## MEDIA path guardrail

Windows absolute paths use the `<drive>:/\1` form (no leading slash). The `/<drive>:/\1` form is forbidden.
A gateway log line "Skipping unsafe MEDIA directive path" means the path has one extra `/`.
