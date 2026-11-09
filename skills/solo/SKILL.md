---
name: solo
description: |
  SOLO v2.1 — TAFA tri-partite separation of powers, engineered in practice. The Rule layer sets rules but does not execute; the Execution layer executes but does not supervise; the Audit layer supervises but does not execute. A three-layer interlocking loop, native to host runtime.
version: 2.1.0
metadata:
  runtime:
    emoji: ⚡
    requires: {}
---

# SOLO v2.1 ⚡ — TAFA Tri-Partite Separation of Powers, Engineered in Practice

> **One person, an entire agent army.**
> The Rule layer sets the rules, the Execution layer does the work, the Audit layer handles auditing. Those who make rules do not execute them; those who execute do not supervise; those who supervise do not take part in execution.
> **The 655 system (6 iron laws + 5 principles + 5 MEV stages) belongs entirely to the Rule layer — it is a rule set, not the whole of SOLO.**

---

## Architecture Overview: A Three-Layer Interlocking Loop

```
┌───────────────────────────────────────────────────────────────────┐
│              1. RULE LAYER — rule-making                          │
│                                                                   │
│  655 system: 6 iron laws (SOUL.md) + 5 principles + 5 MEV stages  │
│  Suit gate · tool guardrails · search fallback chain              │
│                                                                   │
│  Makes the rules, but never takes part in any execution or audit. │
│  Rules are appended, never modified; audit proposals are approved │
│  first, then appended.                                            │
│              │                          ▲                         │
│    rules issued as constraints          │ audit feedback ·        │
│              ▼                          │ rule evolution          │
├───────────────────────────────────────────────────────────────────┤
│              2. EXECUTION LAYER — task execution                  │
│                                                                   │
│  cron / skill pipelines / sessions_spawn                          │
│  MSF · DI · HI · SI · babata-browser · babata-search              │
│                                                                   │
│  Strictly executes the Rule layer's rules (Suit gate self-check   │
│  plus iron-law self-check), but may not change a single one.      │
│  Execution output is left to the Audit layer; no self-review.     │
│              │                                                    │
│    output handed over for audit                                   │
│              ▼                                                    │
├───────────────────────────────────────────────────────────────────┤
│              3. AUDIT LAYER — independent audit                   │
│                                                                   │
│  skills/solo-audit/ — exclusive to a Pro-tier model               │
│  Read-only tools (read / read-only exec / cron runs)              │
│  Proposals only, never execution · five-table benchmark of iron   │
│  laws + MEV + principles + file structure                         │
│                                                                   │
│  Audits the Execution layer, but takes no part in execution.      │
│  Audit output → proposal → Rule-layer approval → rule evolution.  │
│  The Audit layer is itself bound by the Rule layer's              │
│  "self-restraint iron law" and cannot expand its own powers.      │
└───────────────────────────────────────────────────────────────────┘
```

### How the interlocking loop runs

```
  Rule layer ──rules issued──→ Execution layer ──output──→ Audit layer ──proposals──→ Rule layer
  (rule-making)                (execution)                 (supervision)             (evolution)
```

**The three layers do not cross:** the Rule layer does not write the Execution layer's cron payloads, the Execution layer does not modify the Rule layer's iron laws, and the Audit layer does not touch execution tools. Each layer does only its own job.

---

## I. Rule Layer — Rule-Making, Not Execution

### 1.1 The 655 System

| Level | Content | Location | Nature |
|:------|:--------|:---------|:-------|
| 6 iron laws | Privacy / Provenance / Immunity / Authority-responsibility / File economy / Humility | SOUL.md §Core Iron Laws | Right-or-wrong judgment, blocking level |
| 5 principles | Expansion trap (cornerstone) + purity/compression/priority/entropy-reduction (output) | solo-audit SKILL.md §4.3 | Design trade-offs, audit benchmark |
| 5 MEV stages | Suit / Sense / Think / Optimize / Evolve | SOUL.md §MEV Five Layers | Process quality, methodology |

**Note:** 655 is the Rule layer's rule set, not the whole of SOLO. The full form of SOLO is the TAFA three-layer interlocking loop.

### 1.2 Rule change process

```
Audit-layer proposal → owner approval → Rule-layer append → loaded automatically at the Execution layer's next run
                                  ↑
                         Audit layer verifies daily whether it was applied
```

- Rules are appended, never modified (explanations may be added; nothing may be deleted or downgraded)
- The Execution layer may not bypass or downgrade any Rule-layer rule on its own

### 1.3 Ownership files

| File | Responsibility |
|:-----|:---------------|
| SOUL.md | Six iron laws + MEV five layers + identity core |
| AGENTS.md | Workspace rules + skill routing + memory system |
| TOOLS.md | Tool guardrails + search fallback chain + anti-bot rules |
| .sop/ | Standardized SOPs (written only after Rule-layer approval) |

---

## II. Execution Layer — Task Execution, Not Self-Review

### 2.1 Execution skill pipelines

| Skill | Trigger | Output |
|:------|:--------|:-------|
| AI-industry hotspot watch (cron-ai-hotspot) | cron 04:00 | Technology and business signal report |
| Hospital smart supervision (cron-hospital) | cron 06:00 | Healthcare policy supervision intelligence |
| AI medical cases (cron-aimed) | cron 09:00 | AI + healthcare company cases |
| Monthly regulation inspection | cron 10:00 on the 1st of each month | Version check across the whole library |
| MSF | On demand | Medical organization research |
| DI | On demand | Discipline review analysis |
| HI | On demand | Hospital inspection analysis |
| SI | On demand | Inspection and supervision analysis |
| babata-browser | On demand | Anti-bot page capture |
| babata-search | On demand | Chinese-language search |
| deep-research | On demand | Deep research |

### 2.2 Execution-layer iron rules

1. **Before execution:** Suit gate self-check (all six iron laws must pass at Step 0 before proceeding)
2. **During execution:** bound by Rule-layer tool guardrails (search fallback chain · anti-bot rules)
3. **After execution:** output is not self-reviewed — it is left to the Audit layer
4. **Prohibited:** modifying any Rule-layer file · bypassing the Suit gate · self-auditing or self-scoring

### 2.3 Entropy-reduction contract (universal mandatory protocol)

> Corresponds to principle 4 of the Rule layer's five principles, "entropy reduction": the main session holds only structured summaries, while sub-sessions carry the concrete work.
> **This contract applies to every skill or task that uses sessions_spawn, not only SI/DI/HI.**

#### Rule 1: Compressed sub-session hand-back (all sessions_spawn protocols)

```
DONE <output file path>
Summary: ≤3 lines
Anomaly: ⚠️ yes/no
```

**Never read the full text of a sub-agent's output file in the main session.** The main session records only: completion status, file path, a 3-line core summary, and the anomaly flag.

#### Rule 2: Compressed file verification

After receiving a sub-agent's DONE, file verification **checks existence only, not content**:
```
read <output_file_path> → check existence only (size > 0 is enough)
file exists → record the path
file missing → mark failed → report the anomaly
```

#### Rule 3: Mandatory sub-session isolation for draft/analysis stages

Any stage that involves writing reports, working papers, outlines, or analysis documents **must spawn a sub-agent**, and may not be run in the main session:
```
Main session only: spawn → wait for Done → verify the file exists → move to the next stage
Sub-agent: read the input files → analyze/write → write the output file
```

#### Rule 4: Main-session health self-check

Before every sessions_spawn, check the main session's token level:
- **≤ 40%** → run normally
- **40-80%** → automatically trim the hand-back records of closed sub-sessions
- **> 80%** → the whole pipeline must run under sessions_spawn isolation; the main session does no substantive work

How to check: read session_status → Context utilization percentage

#### Exception list

The following may be done directly in the main session:
1. IMA upload (Phase 7 type: direct call from the main session)
2. Final delivery summary (≤3 lines)
3. Pipeline anomaly report (only for FAIL/HUMAN_ESCALATION)
4. Single-step tool calls (search/OCR/crawl and other L1 tasks)

#### Flexibility rules (based on the 655 "priority" principle: safety > accuracy > purity > efficiency)

**1. Retrieval on demand (to prevent over-reduction of entropy)**

A sub-agent may be **explicitly authorized** in its spawn prompt to read upstream raw data (for example, the full JSON output by a Scout); this does not violate the entropy-reduction contract.

Rules:
- The main session still does not read full text automatically (the guardrail is unchanged)
- If a spawn declares `input_authority: read_raw_scout_data`, the sub-agent may read the upstream raw file directly
- The sub-agent's own hand-back must still be compressed (DONE + path + 3 lines)

**2. Progress hand-back (to prevent black-box execution)**

A sub-agent's hand-back may include progress information so the main session can answer "what stage are we at?":
```
DONE <path>
Summary: ≤3 lines
Anomaly: ⚠️ yes/no
Progress: current stage/total stages, estimated remaining
```

**3. Adjustable pipeline depth (to prevent over-layering)**

For medium-complexity tasks (such as company analysis or industry research), stages may be merged after assessment to reduce sub-agent layers:
- Scouts (unchanged; 3 parallel routes work well)
- Analyze + Draft may be merged into one agent (saves one spawn overhead)
- Review (kept; the quality-control guardrail is unchanged)

Assessment criteria:
- How many independent perspectives does the task need? → many perspectives means keep the Analyze layer
- How large is the raw data? → large data means keep the Scout layer
- How strict are the fault-tolerance requirements? → low tolerance means keep the Review layer

#### Consequences of violation

The entropy-reduction contract is audited daily by the solo-audit layer. Violations are recorded as `[ENTROPY_VIOLATION]` and included in audit proposals.

---

## III. Audit Layer — Independent Audit, Not Execution

### 3.1 Audit model

**Independent skill:** `skills/solo-audit/SKILL.md`

| Attribute | Rule |
|:----------|:-----|
| Model | Pro-tier (mandatory) |
| Tools | Read-only allowlist (write/edit/create_media/sessions_send prohibited) |
| Output | Proposals only; never on the execution path |
| Cadence | Daily at 22:00 (SOLO daily audit cron) |

### 3.2 Five-table benchmark

| Benchmark table | Source | What is audited |
|:----------------|:-------|:----------------|
| 6 iron laws | SOUL.md | Right or wrong — was it done? |
| 5 MEV layers | SOUL.md | Process quality — was it done well? |
| 5 principles | solo-audit §4.3 | Design trade-offs — is the design right? |
| Version alignment | PLUGIN-REGISTRY.md | Fleet sync — are versions consistent? |
| File structure | Workspace | Expansion trap — is anything piling up? |

### 3.3 Audit-layer self-restraint

- The Audit layer reads only and does not write; it only proposes, never executes
- When uncertain, mark [uncertain]; do not infer
- Do not widen the audit scope; do not exceed authority
- The Audit layer is itself bound by the Rule layer's "self-restraint iron law"

---

## IV. The Three-Layer Interlock in Detail

### 4.1 Interlock rule matrix

| From \ To | Rule layer | Execution layer | Audit layer |
|:----------|:----------:|:---------------:|:-----------:|
| **Rule layer** | — | Issues 655 rules as constraints | Receives audit proposals → evolves |
| **Execution layer** | Executes Rule-layer rules | — | Accepts audit → remediates |
| **Audit layer** | Outputs audit proposals | Reviews execution output | — |

### 4.2 Consequences of breaking the interlock

| Violation pattern | Example | Consequence |
|:------------------|:--------|:------------|
| Execution layer self-audits | Writing its own audit report after execution | Breaks "execute without supervising"; flagged by the Audit layer |
| Execution layer changes rules | Bypassing the Suit gate to write regulation analysis directly | Breaks "execute without making rules"; task blocked |
| Audit layer takes part in execution | Editing files while auditing | Breaks "supervise without executing"; proposal voided |
| Rule layer takes part in execution | Writing a cron payload inside SOUL.md | Breaks "make rules without executing"; should be split out |
| **Any layer changing rules beyond its authority** | **Editing a cron payload directly without 655 comparison + approval** | **Breaks "authority and responsibility kept separate"; recorded by audit as `[UNAUTHORIZED_MODIFICATION]`** |

#### Rule-change firewall (all three layers · run before every modification)

Before any layer modifies an existing rule, configuration, or cron payload, it must run a three-step check:

```
Step 1: 655 comparison — which principle does this change map to? Is it already covered? Is it an expansion trap?
        Checklist: (1) is this need already covered by an existing rule  (2) does the new rule replace an elimination path  (3) is the new rule orthogonal and non-overlapping
Step 2: Authority check — does my layer have the right to make this change?
        Rule layer → may only append rules, never write execution logic
        Execution layer → may only execute per the rules, never change them
        Audit layer → may only propose, never change directly
Step 3: Owner approval — show the change before modifying; execute only after approval
```

A violation counts as `[UNAUTHORIZED_MODIFICATION]` and is recorded and pushed in the daily audit.

### 4.3 Loop assurance

```
  Rule layer makes rules
    → the Suit gate forces every Execution-layer start to load them
    → the Execution layer executes per the rules
    → execution output is written to the audit repository
    → the Audit layer reads the audit repository daily
    → the Audit layer writes audit proposals to memory/audit/archive/
    → audit proposals are approved by the owner
    → approved rule changes are appended to the Rule layer
    → back to step one
```

---

## V. Operator Directory

| File | Layer | Purpose |
|:-----|:-----:|:--------|
| SOUL.md · AGENTS.md · TOOLS.md | Rule layer | Rule-making and tool guardrails |
| .sop/ | Rule layer | Standardized SOPs |
| skills/solo/SKILL.md | Meta | This document (description of the TAFA architecture) |
| skills/solo-audit/ | Audit layer | Independent audit skill |
| skills/msf/ · di/ · hi/ · si/ | Execution layer | Task pipelines |
| skills/babata-browser/ · babata-search/ | Execution layer | Search and capture toolchain |
| skills/deep-research/ · party-review/ | Execution layer | Analysis and review pipelines |
| memory/audit/ | Audit layer | Audit repository (archive/knowledge/meta) |
| memory/session-logs/ | — | Deprecated (removed in cron22 Step 9) |

---

## Changelog

| Version | Date | Change |
|:--------|:----:|:-------|
| **2.1.0** | **06-11** | **Rebuilt as the TAFA three-layer interlocking loop. Rule layer (655 system) + Execution layer (execution) + Audit layer (audit), each with its own role. Added the interlock rule matrix, violation consequences, and loop assurance. Zero scripts, fully native.** |
| 2.0.3 | 06-07 | Terminology replaced with tri-partite separation of powers (TAFA) |
| 2.0.2 | 06-07 | Added the entropy-reduction contract (Execution-layer protocol) |
| 2.0.1 | 05-24 | Added the pipeline status protocol |
| 2.0.0 | 05-23 | TAFA rewrite |
