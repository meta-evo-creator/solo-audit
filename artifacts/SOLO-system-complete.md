---
name: solo
description: |
  SOLO v2.1 — TAFA three-power architecture, engineering practice. The Rule layer sets rules but does not execute; the Execution layer executes but does not supervise; the Audit layer supervises but does not execute. A three-layer interlocking loop, native runtime.
version: 2.1.0
metadata:
  runtime:
    emoji: ⚡
    requires: {}
---

# SOLO v2.1 ⚡ — TAFA Three-Power Architecture · Engineering Practice

> **One person, an entire agent army.**
> The Rule layer sets the rules, the Execution layer does the real work, the Audit layer runs the audit. Those who make rules do not execute; those who execute do not supervise; those who supervise do not take part in execution.
> **655 (6 iron rules + 5 principles + 5 MEV) belongs entirely to the Rule layer — it is the rule system, not the whole of SOLO.**

---

## Architecture Overview: Three-Layer Interlocking Loop

```
┌─────────────────────────────────────────────────────────────────────┐
│                    (1) Rule Layer — rule making                     │
│                                                                     │
│  655 system: 6 iron rules (SOUL.md) + 5 principles + 5 MEV          │
│  Suit gate · tool guardrails · search fallback chain                │
│                                                                     │
│  Sets rules, but never takes part in any execution or audit.        │
│  Rules are appended, never modified; audit proposals are            │
│  approved first, then appended.                                     │
│              │                          ▲                           │
│    rules issued as constraints          │  audit feedback ·         │
│              ▼                          │  rule evolution           │
├─────────────────────────────────────────────────────────────────────┤
│                 (2) Execution Layer — task execution                │
│                                                                     │
│  cron / skill pipelines / sessions_spawn                            │
│  MSF · DI · HI · SI · babata-browser · babata-search                │
│                                                                     │
│  Strictly follows Rule-layer rules (Suit gate self-check + iron     │
│  rule self-check), but has no authority to change any of them.      │
│  Execution output is left for the Audit layer; no self-review.      │
│              │                                                      │
│    output handed to audit                                           │
│              ▼                                                      │
├─────────────────────────────────────────────────────────────────────┤
│                  (3) Audit Layer — independent audit                │
│                                                                     │
│  skills/solo-audit/ — Pro-tier model only                           │
│  Read-only tools (read / exec read-only / cron runs)                │
│  Files proposals only, never executes · benchmarks five tables:     │
│  iron rules + MEV + principles + file structure                     │
│                                                                     │
│  Audits the Execution layer but does not take part in execution.    │
│  Audit output → proposal → Rule-layer approval → rule evolution.    │
│  The Audit layer is itself bound by the Rule layer's restraint      │
│  iron rule and may not expand its own powers.                       │
└─────────────────────────────────────────────────────────────────────┘
```

### How the Interlocking Loop Runs

```
  Rule Layer ──issues rules──→ Execution Layer ──output──→ Audit Layer ──proposals──→ Rule Layer
  (makes rules)                (executes)                  (supervises)              (evolves)
```

**The three layers never overlap:** the Rule layer does not write the Execution layer's cron payloads, the Execution layer does not modify the Rule layer's iron rules, and the Audit layer does not touch execution tools. Each layer does only its own job.

---

## 1. Rule Layer — Sets Rules, Never Executes

### 1.1 The 655 System

| Level | Content | Location | Nature |
|:-----|:-----|:-----|:-----|
| 6 iron rules | Privacy / provenance / immunity / authority-responsibility / cherish files / humility | SOUL.md §core iron rules | Right-or-wrong judgment, blocking level |
| 5 principles | Expansion trap (cornerstone) + purity / compression / priority / entropy reduction (output) | solo-audit SKILL.md §4.3 | Design trade-offs, audit benchmarks |
| 5 MEV | Suit / Sense / Think / Optimize / Evolve | SOUL.md §MEV five levels | Process quality, methodology |

**Note:** 655 is the Rule layer's rule set, not the whole of SOLO. The complete form of SOLO is the TAFA three-layer interlocking loop.

### 1.2 Rule Change Process

```
Audit proposal → owner approval → Rule-layer append → Execution layer loads it automatically on its next run
                                  ↑
                         Audit layer verifies daily whether it was executed
```

- Rules are appended, never modified (explanations may be added; nothing may be deleted or downgraded)
- The Execution layer may not bypass or downgrade any Rule-layer rule on its own

### 1.3 Ownership Files

| File | Responsibility |
|:-----|:------|
| SOUL.md | Six iron rules + MEV five levels + identity core |
| AGENTS.md | Workspace rules + skill routing + memory system |
| TOOLS.md | Tool guardrails + search fallback chain + anti-scraping rules |
| .sop/ | Standardized SOPs (written only after Rule-layer approval) |

---

## 2. Execution Layer — Runs Tasks, Never Self-Audits

### 2.1 Execution Pipelines

| Skill | Trigger | Output |
|:-----|:---------|:-----|
| AI industry hotspot monitoring (cron-ai-hotspot) | cron 04:00 | Technology and business signal report |
| Hospital smart supervision (cron-hospital) | cron 06:00 | Healthcare policy supervision intelligence |
| AI healthcare cases (cron-aimed) | cron 09:00 | AI + healthcare company cases |
| Monthly regulation sweep | cron, 1st of each month 10:00 | Full-library version check |
| MSF | on demand | Medical organization research |
| DI | on demand | Discipline review analysis |
| HI | on demand | Hospital inspection analysis |
| SI | on demand | Inspection and supervision analysis |
| babata-browser | on demand | Anti-scraping page capture |
| babata-search | on demand | Chinese-language search |
| deep-research | on demand | Deep research |

### 2.2 Execution-Layer Iron Rules

1. **Before execution:** Suit gate self-check (Step 0 — all six iron rules must pass before release)
2. **During execution:** bound by Rule-layer tool guardrails (search fallback chain · anti-scraping rules)
3. **After execution:** do not self-review the output — leave it to the Audit layer
4. **Prohibited:** modifying any Rule-layer file · bypassing the Suit gate · self-auditing or self-scoring

### 2.3 Entropy-Reduction Contract

> Corresponds to the Rule layer's principle (4) "entropy reduction": the main session stores only structured summaries; sub-sessions carry the concrete task.

**Sub-session return format (all sessions_spawn protocols):**
```
Summary: ≤3 lines
Output: concrete artifact path
Anomaly: ⚠️ yes/no
```

**Main-session health thresholds:**
- ≤ 40% → normal
- > 40% → automatically trim returns from closed sub-sessions
- > 80% → force isolated execution via session_spawn

---

## 3. Audit Layer — Audits Independently, Never Executes

### 3.1 Audit Model

**Standalone skill:** `skills/solo-audit/SKILL.md`

| Attribute | Rule |
|:-----|:------|
| Model | Pro-tier (mandatory) |
| Tools | read-only allowlist (write/edit/create_media/sessions_send prohibited) |
| Output | proposals only, never on the execution path |
| Cadence | daily 22:00 (SOLO daily audit cron) |

### 3.2 Five-Table Benchmarking

| Benchmark table | Source | What it audits |
|:-------|:-----|:---------|
| Six iron rules | SOUL.md | Right or wrong — was it done? |
| MEV five levels | SOUL.md | Process quality — was it done well? |
| Five principles | solo-audit §4.3 | Design trade-offs — is the design right? |
| Version alignment | PLUGIN-REGISTRY.md | Fleet sync — are versions consistent? |
| File structure | workspace | Expansion trap — is anything piling up? |

### 3.3 Audit-Layer Restraint

- The Audit layer reads but never writes; it files proposals and performs no execution
- When uncertain, mark [uncertain]; never infer
- Do not widen the audit scope or exceed its authority
- The Audit layer is itself bound by the Rule layer's "restraint iron rule"

---

## 4. The Three-Layer Interlock in Detail

### 4.1 Interlock Matrix

| From \ To | Rule Layer | Execution Layer | Audit Layer |
|:------|:----:|:----:|:----:|
| **Rule Layer** | — | issues 655 rules as constraints | receives audit proposals → evolves |
| **Execution Layer** | executes Rule-layer rules | — | accepts audit → remediates |
| **Audit Layer** | outputs audit proposals | audits execution output | — |

### 4.2 Consequences of Breaking the Interlock

| Violation pattern | Example | Consequence |
|:---------|:-----|:-----|
| Execution layer self-audits | writing its own audit report after execution | violates "execute, do not supervise"; the Audit layer raises it as a finding |
| Execution layer changes rules | bypassing the Suit gate to write regulatory analysis directly | violates "execute, do not make rules"; task blocked |
| Audit layer takes part in execution | editing files while auditing | violates "supervise, do not execute"; the proposal is void |
| Rule layer takes part in execution | writing a cron payload into SOUL.md | violates "make rules, do not execute"; it must be stripped out |

### 4.3 Loop Guarantees

```
  Rule layer makes rules
    → the Suit gate forces every Execution-layer start to load them
    → the Execution layer executes per the rules
    → execution output is written to the audit repository
    → the Audit layer reads the audit repository every day
    → the Audit layer writes audit proposals to memory/audit/archive/
    → audit proposals are approved by the owner
    → approved rule changes are appended to the Rule layer
    → back to step one
```

---

## 5. Operator Directory

| File | Layer | Purpose |
|:-----|:------:|:-----|
| SOUL.md · AGENTS.md · TOOLS.md | Rule | Rule making and tool guardrails |
| .sop/ | Rule | Standardized SOPs |
| skills/solo/SKILL.md | Meta | This document (TAFA architecture description) |
| skills/solo-audit/ | Audit | Independent audit skill |
| skills/msf/ · di/ · hi/ · si/ | Execution | Task pipelines |
| skills/babata-browser/ · babata-search/ | Execution | Search and capture toolchain |
| skills/deep-research/ · party-review/ | Execution | Analysis and review pipelines |
| memory/audit/ | Audit | Audit repository (archive/knowledge/meta) |
| memory/session-logs/ | — | Deprecated (removed from cron22 Step 9) |

---

## Changelog

| Version | Date | Change |
|:-----|:----:|:------|
| **2.1.0** | **06-11** | **TAFA three-layer interlocking loop rebuild. Rule layer (655 system) + Execution layer (execution) + Audit layer (audit), each with its own job. Added the interlock matrix + violation consequences + loop guarantees. Zero scripts, purely native.** |
| 2.0.3 | 06-07 | Terminology replaced with the three-power separation (TAFA) |
| 2.0.2 | 06-07 | Added the entropy-reduction contract (Execution-layer protocol) |
| 2.0.1 | 05-24 | Added the pipeline status protocol |
| 2.0.0 | 05-23 | TAFA rewrite |
