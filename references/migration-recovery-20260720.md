# host runtime → Hermes Migration Loss Recovery Log

> 2026-07-20 | the agent
> Every platform migration is inevitably accompanied by knowledge loss. This document records all critical assets and lessons recovered from host runtime.

## Recovered files

| File | Path | Content |
|:-----|:-----|:-----|
| HEARTBEAT.md | hermes/ | Fleet monitoring, file budget, daily audit trigger |
| MEMORY.md | hermes/ | WIKI uniqueness guardrail, audit sync rules, shared supervision config |
| TOOLS.md | hermes/ | Tavily dual-key rotation, search degradation chain, anti-scraping rules |
| PLUGIN-REGISTRY.md | hermes/ | Last-trigger time per skill |
| FLEET-CLUSTER.md | hermes/ | Three-axis skill classification |
| IDENTITY.md | hermes/ | Identity core |
| constraint-efficiency-balance.md | memory/goals/ | Long-term goal tracking |
| 2026-07-04-findings.json | memory/audit/findings/ | Audit findings (expansion trap) |
| SP-001-rejected.md | memory/audit/proposals/ | Rejected proposal record |
| SOLO system update v5.1 | memory/system/ | Key methodology upgrade |
| forum-collaboration-sop.md | memory/system/ | Forum collaboration SOP |
| workspace-file-management.md | memory/system/ | WIKI write five guards |

## Recovered crons

| cron | Description |
|:-----|:-----|
| Weekly regulation check (Mondays 08:00) | Version check of 5 core regulations, complementary to the monthly check |
| Daily temp-file cleanup (03:00) | /tmp/, desktop leftovers, delegation cache |

## Hardened guardrails

| Guardrail | File | Change |
|:-----|:-----|:-----|
| Anti-hallucination hard constraint | search-rg.md | Clause numbers must be extracted with rg; memory is forbidden |
| Subject-element gate | search-rg.md | Step 1.5 restored |
| Citation <80% REJECT | agent4-review.md (MSF) | Code-level grep |
| Humility check | agent4-review.md (MSF+DR) | Automatic REJECT |
| Human Gate <65 | MSF+DR+DI SKILL.md | OPL human-factor gate |

## Core lessons

1. **In-memory recall ≠ file fact** — always read the source file, never rely on training memory
2. **"If the knowledge base has it, look before you use it"** — the inspection working-paper lesson applies to every skill
3. **rg no-match ≠ regulation does not exist** — the full degradation chain must be followed
4. **When a problem appears, check execution first; do not add rules** — the core finding of AUDIT-20260704
5. **Changing cron config must sync the prompt version number** — 2026-06-27 lesson
