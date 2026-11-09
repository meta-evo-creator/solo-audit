# 755 Five-Principles Guardrail Implementation Standard

## Expansion trap (cornerstone, highest priority)
Before modifying a core file, the add/delete volume must be declared first. No declaration before editing → 🔴 blocked.
- SOUL.md / AGENTS.md / MEMORY.md / skill SKILL.md / agent .md are all subject to this constraint
- Declaration format: add [N] lines + delete [M] lines + net change [N-M] + reason
- Net change >0 with no corresponding deletion → blocked

## Compression
The same information must not be repeated across 3+ files. Before adding, search for an existing synonymous statement.

## Entropy reduction
Main-session context >80% → compress first, then continue. Memory >80% → proactively merge fragments (do not add; compress what already exists first).
Hermes delegate_task natively handles subagent summaries; no additional constraint.

## Priority
P0 tasks (functional fixes / compliance remediation / audit proposals) must not be blocked by lower-priority tasks. A P0 proposal not executed within 72h → audit-layer escalation.

## Purity
Fabrication is forbidden in output: timestamps ("afternoon edition", "15:00 CST"), tool status ("Tavily exhausted"), version numbers ("MEV v7.3").
When writing a report, do not attach a time, do not declare an engine, do not describe the collection process. Unverified metadata → violation of Iron Rules 6 and 7.

## DI routing search guardrail
For the quick/full/interview routes, 1a ∥ 1b → 1c → 2 (WIKI + legal database) cannot be skipped.
Regulatory consultation ≠ a direct answer — search first, then answer.
Search scope: discipline/regulations/ + medical/ (mandatory dual directory for cases involving medical-institution personnel)
