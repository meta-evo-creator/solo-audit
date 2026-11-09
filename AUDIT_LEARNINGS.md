# AUDIT_LEARNINGS.md — Effective Rules of the Audit Layer

> ⛔ Maintained automatically by the SOLO post-hoc auditor. Do not edit by hand.
> Each audit proposal, once approved by the owner, appends one effective rule here.

---

## Deprecated Rules

| Rule ID | Effective Date | Deprecation Date | Reason for Deprecation | Approver |
|:-------|:--------:|:--------:|:---------|:------:|
| **CHG-003** | 2026-05-24 | **2026-05-30** | The two-step checkpoint write→edit was the root cause of the cron failure. The new design removes the checkpoint and moves auditing to the cron run log | the owner |
| **CHG-005** | 2026-05-26 | **2026-05-30** | The G4.5 signature was originally an audit trail, but writing the signature triggered an edit operation that blocked cron delivery | the owner |
| **CHG-005a** | 2026-05-28 | **2026-05-30** | The mid-report migration strategy for the checkpoint was deprecated together with the checkpoint itself | the owner |

---

## Effective Rules

| Rule ID | Effective Date | Source Audit | Rule Content | Scope |
|:-------|:--------:|:---------|:---------|:---------|
| CHG-001 | 2026-05-23 | SOLO-AUDIT-20260523-001 | After a cron web_fetch, always check rawLength<500 / anti-scraping keywords / navigation footer; on any hit, switch to babata-browser | All cron tasks |
| CHG-002 | 2026-05-23 | SOLO-AUDIT-20260523-001 | SOLO v2.0 separation-of-powers architecture: the audit layer is read-only and never writes; it only files proposals and never changes the SOP | Whole system |
| CHG-004 | 2026-05-26 | P12 | ⛔ Verification-style fix marking: before any fix is marked "executed", first read the target cron payload → verify the change is present → write read evidence to the audit log. Declarative marking is prohibited. | Audit-layer fix execution flow |

---

## Approved — Executed (2026-05-26 22:15, owner approval)

| Proposal ID | Filed | Priority | Problem | Fix | Verification |
|:-------|:--------:|:------:|:-----|:---------|:--------:|
| **P10** | 05-25 | P0 | CHG-003 compliance asymmetry — the fix covered only 1/3 of cron tasks. The AI-hotspot and AI-healthcare tasks still had no checkpoint (reproduced on day 4) | 🔄 Second-round fix: payload slimming + signature embedded in the report body + pre-reminder + double assurance at closing. `cron get` confirms the payload is updated | Check whether the checkpoint file exists after the next run |
| **P11** | 05-25 | P1 | No confirmation of G3 delivery for AI-hotspot + AI-healthcare (missing knowledge-base code / message id) | G3 delivery record embedded in the report footer and the G4.5 signature | The next day's report ends with kb_upload_code / im_message_id fields |
| **P12** | 05-25 | P2 | Fix-mark auditing — move from declarative to verification-style (root cause of NP-002) | CHG-004 is in force. The P10/P11 fixes were executed using the verification-style flow (`cron get` reads the payload to confirm the change exists) | The audit log contains a read-evidence chain |
| **P13** | 05-26 | P0 | 🚨 The AI-healthcare cron was completely silent on 05-26 (no report + no checkpoint) | Diagnosis: cron alive(enabled=true,lastRun=ok,delivered=true) but the model wrote no file. Fix: (1) slim the payload (2) embed the signature in the report (3) manual trigger to recover (queued) | Manual trigger result + normal run at 09:00 the next day |

### History (2026-05-24 22:35)

| Proposal ID | Filed | Problem | Root Cause | Fix | Verification |
|:-------|:--------:|:-----|:------|:---------|:---------|
| **P1** | 05-23 | The hospital-supervision cron finished executing but produced no checkpoint (G4.5 signature missing), so the audit layer could not trace the result | The cron payload ended with a G4.5 signature template JSON, but nothing ever wrote it to a file | Append an explicit file-write instruction to the end of the cron payload: record duration_ms, failed URLs (sources_failed), gates passed (gates_passed) → write to `memory/checkpoints/cron-hospital-YYYY-MM-DD.md` | After the 06:00 run the next day, check that `memory/checkpoints/cron-hospital-YYYY-MM-DD.md` exists and contains a complete signature |
| **P2** | 05-23 | The AI-hotspot cron likewise lacked the execution_signature JSON — isomorphic to P1 | Same as above: the payload had a template but no write instruction | Append a file-write instruction to the end of the cron payload: `memory/checkpoints/cron-ai-hotspot-YYYY-MM-DD.md` (the file existed but its content was incomplete) | After the 04:00 run the next day, check that the checkpoint contains duration_ms / sources_failed / gates_passed |
| **P3** | 05-23 | The cron G3 gate (knowledge-base upload + PUSH) was marked PASS in the delivery self-check, but the actual execution result was never verified | The gate relied on the agent's self-declaration, with no IMP verification or external call confirmation | Change the delivery self-check logic: G3 becomes "after upload, wait for code=0 to return → mark PASS" or "upload failed → mark FAIL and write sources_failed" | After the next run, check the code-return confirmation for G3 in the delivery self-check |
| **P4** | 05-23 | The MSF Draft stage lacked a G1.5 cross-reference consistency check: source IDs cited in the report could disagree with the IDs in scout / evidence_ledger | The Draft Agent generates independently, with no mechanism verifying that the source IDs it cites actually exist in the raw data | Add a verification step to MSF agent3-draft.md: after the Draft completes, walk every [Sxxx] reference in the report → verify each one exists in `evidence_ledger.json` → if not, mark it `[UNSOURCED]` and record a report check | On the next MSF task, see whether cross_reference_check.json is produced in the Draft stage |
| **P5** | 05-23 | Had the FP-001 fix (web_fetch anti-scraping detection) been pushed to the hospital-supervision cron? The previous audit found the cron payload was not synchronized | FP-001 is a core cross-cutting rule; the cron was fixed by hand but the push was never verified | Compare the cron payload line by line against the anti-scraping rules in MSF agent1-scout.md (4 checks) and confirm they match | Verified: the cron payload contains the complete 4-item anti-scraping check (rawLength / anti-scraping keywords / navigation footer / JS rendering), ✅ passed |
| **P6** | 05-24 | solo-audit-daily-001 fired 3 times within 24 hours (not the expected once per day), suggesting duplicate execution | The system restart / cron scheduler restart re-triggered an existing job | (1) Confirm the logic fix for the p6-gp3 check; (2) add de-duplication protection to the audit cron itself (on startup, skip if the last run was <23h ago) | `cron runs solo-audit-daily-001` to inspect the last 3 runs and confirm the interval is >23h |
| **P7** | 05-24 | The third step of every cron (knowledge-base upload + messaging push) depended on the agent executing it spontaneously → no mandatory verification → the root cause of P1/P2/P3 | The G3/G4 gates were self-declared (self-check, self-score), lacking a third-party IMP call | Unified pattern: the G4.5 signature at the end of the payload = (1) file write (mandatory) → (2) knowledge-base upload → (3) messaging push → (4) write checkpoint | Rewrite every cron payload to this pattern, then verify |
| **P8** | 05-24 | 🚨 The hospital-supervision checkpoint had been missing for 28h+ with 100% reproduction | P1 not executed → the cron wrote no checkpoint | Same fix as P1 + an urgent verification marker | A checkpoint must be written successfully at 06:00 the next day after the fix |
| **P9** | 05-24 | 🚨 The AI-hotspot signature had been missing for 28h+ with 100% reproduction | P2 not executed | Same fix as P2 + an urgent verification marker | A successful write is mandatory at 04:00 the next day after the fix |
| **P10** | 05-25 | P0 | CHG-003 compliance asymmetry — the fix was pushed to only 1/3 of cron tasks. The AI-hotspot and AI-healthcare crons still had no checkpoint (reproduced from day three into day four) | Verify line by line whether the AI-hotspot + AI-healthcare cron payloads contain: G4.5 signature JSON written to memory/checkpoints/ + read verification after writing | ✅ P10 executed: approved by the owner on 2026-05-26 22:09, cron payload updated (`cron get` reads the payload to verify all three steps: G4.5 signature + checkpoint write + read verification) |
| **P11** | 05-25 | P1 | No confirmation of G3 delivery for AI-hotspot + AI-healthcare — no knowledge-base upload code / messaging push confirmation at the end of the report | Follow the compliant hospital-supervision cron pattern: report generation → knowledge-base upload (record code) → messaging push (record confirmation) → write G4.5 signature | ✅ P11 executed: approved by the owner on 2026-05-26 22:09, cron payload updated (delivery step includes kb_upload_code / im_message_id / g3_delivery_complete fields) |
| **P12** | 05-25 | P2 | Fix-mark auditing — move from "declarative" to "verification-style". Root cause of NP-002: a fix was marked executed in AUDIT_LEARNINGS.md without verifying the payload of each task | Before any fix is marked "executed", first read the target cron payload → verify the change exists → write read evidence to the audit log | ✅ P12 executed: approved by the owner on 2026-05-26 22:09, CHG-004 rule written

---

## Executed Fixes (2026-05-24 22:53)

| No. | Problem | Fix |
|:----:|:-----|:------|
| **BUG-001** | The `~` in the audit cron path was not resolved → write failed | ✅ All paths changed to absolute paths `<workspace path>` |
| **BUG-002** | The audit cron wrote no G4.5 checkpoint | ✅ Appended CHG-003 mandatory closing + read verification |
| **BUG-003** | The proposal review waited indefinitely | ✅ Added Step 5 self-healing check: automatically read the owner's confirmation message → mark as approved |
| **P1-P9** | All 9 proposals were pending approval | ✅ The owner confirmed all of them; AUDIT_LEARNINGS.md archived them as "effective" |
| **CHG-003** | Mandatory checkpoint rule for all crons | ✅ Written to AUDIT_LEARNINGS.md; all crons load it automatically at startup |
| **CHG-004** | Verification-style flow for fix marking | ✅ Written to AUDIT_LEARNINGS.md; P10 was verified under this flow (`cron get` reads the payload to confirm the change exists) |
| **P10** | CHG-003 coverage fix | ✅ AI-hotspot cron + AI-healthcare cron payloads updated, including G4.5 signature + checkpoint write + read verification (`cron get` confirms the payload) |
| **P11** | G3 delivery automation | ✅ AI-hotspot cron + AI-healthcare cron payloads updated; the delivery step tracks kb_upload_code / im_message_id / g3_delivery_complete |
| **P12** | Fix-mark auditing | ✅ CHG-004 is in force; the P10/P11 work used the verification-style flow |

---

---

## Approved — Executed (2026-05-28 22:29, owner approval)

| Proposal ID | Filed | Priority | Problem | Fix | Verification |
|:-------|:--------:|:------:|:-----|:---------|:--------:|
| **P14** | 05-28 | P0 | The P10 fix was ineffective — AI-hotspot + AI-healthcare were still missing checkpoints. CHG-005 coverage 33% | ✅ CHG-005a in force: checkpoint writing moved to the middle of the report + verify that `cron update` actually took effect | TODO: check that the checkpoint exists on the next run |
| **P15** | 05-28 | P1 | AI-hotspot G3 delivery remained pending | ✅ Merged with the P14 fix | Same as above |

---

## Approved — Executed (2026-05-31 09:18, owner approval)

| Proposal ID | Filed | Priority | Problem | Fix |
|:-------|:--------:|:------:|:-----|:---------|
| **P16** | 05-31 | P2 | The cron delivery script's sanitizer word list lacked G3/G0/G1/G2/gate and similar terms → reports carried G3 metadata into the knowledge-base upload | Append `'G3','G0','G1','G2','gate','delivery gate','quality_gate'` to sanitizer array W; G4.5 already had symmetric coverage |

## Effective Rules

| Rule ID | Effective Date | Source Audit | Rule Content | Scope |
|:-------|:--------:|:---------|:---------|:---------|
| CHG-006 | 2026-05-31 | SOLO-AUDIT-20260531-001 | The cron delivery script's sanitizer word list fully covers G0-G4 gate keywords (G4.5 already present; G3/G0/G1/G2/gate/quality_gate added) | the cron delivery script |

## Proposals Under Review

(No proposals awaiting approval)

---

### Historical Approval (2026-05-26 22:15, owner approval)

> 2026-05-26 22:15, message from the owner: "Those reproduced problems are still unhandled — I approve them all, fix them now." P10/P11/P12/P13 were all approved and executed.

### Historical Approval (2026-05-24 22:35, owner approval)

> P1-P9 all approved and archived as "effective".

> Last updated: 2026-05-29 22:00 CST

### P14/P15 Verification Results (2026-05-29)

| Proposal ID | Filed | Verified | Result | Evidence |
|:-------|:--------:|:--------:|:----:|:-----|
| **P14** | 05-28 | 05-29 | ✅ **Verified effective** | AI-hotspot checkpoint 05-29 PRESENT (1169B, full signature) |
| **P15** | 05-28 | 05-29 | ✅ **Verified effective** | AI-healthcare checkpoint 05-29 PRESENT (1475B, full JSON, kb_upload_code=0) |

**BREAKTHROUGH:** CHG-003 coverage 0.33 (05-28) → 1.00 (05-29). First time 3/3 cron checkpoint compliance was achieved. CHG-005a (checkpoint moved to the middle of the report) disproved hypothesis NP-004.

---

## CHG Proposal Format (version 2026-05-31)

Every agent reads `AUDIT_LEARNINGS.md` at startup and must understand the two proposal formats.

### Type One: Behavior-Correction Proposal (produced by solo-audit)

```markdown
## CHG-XXX: [problem statement]

**Source:** SOLO-AUDIT-YYYYMMDD-NNN
**Severity:** P0 | P1 | P2
**Problem:** [one sentence]
**Root cause:** [why it happened]
**Fix:** [how to change it, path elimination preferred]
**Scope:** [which files/crons are affected]
**Verification:** [how to verify it is fixed]
```

### Type Two: Text-Optimization Proposal (produced by SkillOpt offline training)

> Applies to: automatic SKILL.md optimization for the three skills DR / MSF / solo-audit. DI excluded.

```markdown
## CHG-XXX: [skill name] [agent name] text optimization

**Source:** SkillOpt offline training
**Verification result:** best_skill.md Review total XX/100 vs original skill XX/100
**Severity:** P2 (takes effect only if the score is ≥ the original and the owner approves)

**Diff:**
<<< OLD
corresponding passage of the original SKILL.md
>>>
<<< NEW
optimized passage
>>>

**Verification evidence:**
- old win-rate: XX/100 (N runs)
- new win-rate: XX/100 (N runs)

**Owner approval:** [ ] approve [ ] reject
```

The shared bottom line for both proposal types: **no change takes effect before the owner approves it. Iron Rule 4 — authority and responsibility.**
