---
name: "solo-audit"
version: 6.13.0
description: "SOLO Audit v6.13 — TAFA-Native + Kanban approval: seven iron rules benchmarking + current five principles (refined 7/25, finalized 8/25) + MEV benchmarking + structural guardrail live testing + trace verification (OBS, scripted). Four-agent pipeline."

metadata:
  hermes:
    emoji: 🔍
    pipeline: delegate_task
    delivery: native_chat
---

# SOLO Audit Agent v6.13

> **v6.14 supplement (2026-11-04, approved by the owner)**: adds Section 7 "Runtime Environment Consistency" (item 24) — `scripts/env_consistency.py` → `audit_collect.py::check_env_consistency`, regression gate `scripts/tests/test_env_consistency.py`. Origin: a TTS service returned 403 for two consecutive days while all 23 items were green, because the static criterion looked at the pyproject pin and the development venv, whereas the runtime actually runs in the package manager's `installs/…/venv` (instance 11 of "criteria detached from reality"). Criteria must trust only the real runtime version plus one real call; an uninstalled optional dependency is not a violation; a missing criterion is always ⚠️ and never ✅.
> **v6.13 supplement (2026-10-03, approved by the owner, card `<card-id>`)**: Section 6 "Trace Verification" changed from **manual model judgement** to **script output** (`scripts/audit_collect.py::check_trace` → `scripts/trace_scan.py`, logic and regression tests from the same source); scan scope adds `workspace/**/io_stage` — the old definition pointed only at `tmp/` and `reports`, so a real case's `_trace.md` sat on disk yet the audit repeatedly reported "0 case tasks" = **false pass** (instance 8 of "criteria detached from reality"). A regression case `scripts/tests/test_trace.py` was added with this fix.
> **v6.12 supplement (2026-09-28)**: two **regression-gate self-check** rules — (1) `scripts/tests/` holds pytest cases only; one-off validation/erratum scripts go in `scripts/checks/`; (2) full regression **must run `pytest <directory>` on the whole directory**; a passing single file is forbidden as regression evidence; before reporting "pass", confirm the gate was actually collected and executed (`no tests ran` = failure). Rationale: LEARNED family "criteria detached from reality", instance 6.
> TAFA full loop: authority layer (SOLO755 Constitution) → capability layer executes → audit layer audits and detects violations → ⏵ routes to capability layer → (1) capability layer proposes → authority layer (Human in the Loop) approves → (1) capability layer fixes → (2) capability layer guardrails (attribute · localize · consolidate · verify) → (3) audit layer verifies next round
> v6.9: **five-principle audit aligned with the authority layer's current version (2026-08-26)** — the old "purity/compression" wording is obsolete (refined to "streamline" on 7/25, "guardrail" added in the 8/25 cognitive finalization) — the principle audit paragraph is corrected to: bloat trap / streamline / priority / entropy reduction / guardrail (guardrail live testing in Section 5).
> **v6.10 supplement (2026-09-23)**: adds Section 6 "Trace Verification (OBS level)" — corresponds to the `supervision-shared` "capability-layer trace standard" (set by the owner, defined by the capability layer); the audit layer only verifies, never sets the standard.
> **v6.9 supplement (2026-09-13, the owner clarified the audit layer's responsibility boundary)**: this audit = **auditing capability-layer compliance against authority-layer requirements**. **Not every concrete fix goes through this audit** — concrete fixes follow the capability layer's normal loop (proposal → approval → execution → guardrail consolidation); the audit only handles "detect violations + route", it neither takes over nor acts on behalf.
> **Hence one self-binding rule (anti-gaming)**: **audit criteria/thresholds belong to the authority layer; they must never be changed to make alerts look better**. Example: a skill-size alert is glaring, so the criterion is switched from bytes to characters to make it vanish — that is **changing the ruler, not the problem**. Only the authority layer may change a criterion; otherwise fix the capability layer's actual state.
> v6.8: **guardrail effectiveness verification** — the audit layer not only detects violations but also verifies whether structural guardrails (Ring 99 user-file protection, audit_hook pre-tool interception, policy-engine) actually take effect. The acceptance test for a TAFA rollout = the guardrail exists, not that everyone behaves.
> v6.7: three-layer attribution corrected — **authority layer = SOLO 755 Constitution (seven iron rules + five principles + MEV five layers)**. All audit-layer agents (COLLECT/VERIFY/REPORT) are internal audit-layer steps; ANALYZE belongs to the capability layer. The authority layer = the owner in person (Human in the Loop), with no agent.
> v6.5: the audit layer is strictly narrowed — detect violations + route only; Kanban card titles "⏳ pending analysis" contain no solution. Proposing is the capability layer's job.
>
> | Level | Flow | Decision basis |
> |:----:|:-----|:-----|
> | 🔴P0 | route to authority layer · human approval | privacy / accountability / data loss / production impact |
> | ⚠️P1 | route to capability layer · automatic proposal · execute after approval | path / format / verification / threshold |
> | 🔵OBS | route to capability layer · record only | performance fluctuation / trend / non-urgent |

---

## 0. Protocol Signature


```yaml
PROTOCOL: solo-audit v6.13
input:   { trigger: 'cron' | 'direct', scope: TAFAScope }
output:  AuditResult | void(no violation)

TAFA three-layer alignment:
  authority layer: SOLO 755 Constitution = seven iron rules + five principles (bloat / streamline / priority / entropy reduction / guardrail) + MEV five layers (Suit/Sense/Think/Optimize/Evolve). Held by the owner (Human in the Loop).
  capability layer: 14 skills + sub-agent pipeline + cron jobs. Executes proposals, fixes, guardrail consolidation.
  audit layer: solo-audit. Audits whether the capability layer obeys authority-layer (SOLO755) rules. Detects + routes only, never decides.
  audit scope: iron-rule benchmarking + principle benchmarking + MEV benchmarking + infrastructure + learning loop
```

## Audit Scope (audit layer · benchmarked against the authority layer SOLO755 Constitution)

> This section states only the **criteria** (what counts as a violation, and at what level). Item-level detail (live commands / tables / wording) has been moved verbatim into
> `references/audit-scope-v613.md` (slimmed 2026-10-26 · card `<card-id>`) — consult that file before doing hands-on checks.

### 1. Iron-Rule Audit
Scan cron logs for violation patterns. (2) provenance (no source >20% ⚠️) | (4) accountability (skipping Ring 2 🚨) | (7) honesty (undeclared degradation 🚨) | (3)(6) routine checks. Cross-day recurrence → escalate to P0.
**Item 4 user-file overreach (dedicated)**: the capability layer overreaching on user files (`<user-profile>`, Desktop/Documents/Downloads/OneDrive, chat / enterprise-chat data) = 🚨P0, **unaffected by any "verbal permission"** (a verbal "clean up / migrate / delete" is not authorization); `[BLOCKED]` records in `logs/audit_hooks.log` must be assessed for whether the capability layer tried repeatedly. Detail → refs Section 1.

### 2. Principle Audit (against the authority layer's current five principles · refined 2026-07-25 · cognitive finalization 2026-08-25)
**Bloat trap** (when a problem appears, first check whether an existing mechanism was executed · a new rule is not the same as removing a path · three questions before adding a rule) | **Streamline** (few but sharp institutions · redundant skills/mechanisms/documents) | **Priority** (grasp the principal contradiction · P0 proposals within 72h · manage high risk first, leave low risk alone) | **Entropy reduction** (main session ≤80% + Memory ≤90% · clear information flow · traceability) | **Guardrail** (structural, not willpower — see live verification in Section 5 · a signboard is not a guardrail).

### 3. MEV Audit
Suit (discipline-inspection tasks executed directly in the main session 🚨) | Sense (skipping the WIKI ⚠️) | Think (skipping the enhancement chain ⚠️) | Optimize (unscored or <60 🚨) | Evolve (lessons not converted ⚠️).

### 4. Agent Performance Evaluation (observation only)
Poor performance is not a violation — record only, never judge. Criteria: hallucination rate >15% ⚠️ | latency >2x baseline ⚠️ | retries >3 ⚠️; results go to `collected_data.json.agent_performance`. Detail → refs Section 4.
Engine + hands heartbeat (search API / legal database / knowledge base / academic index / executor / reviewer) | cron status + skills directory | execution status of the previous round's proposals (>7 days = P0).

### 5. 🔴 Structural Guardrail Verification (new in v6.8 · TAFA rollout acceptance test)
Criterion: **the acceptance test for a TAFA rollout = the guardrail exists, not that everyone behaves** — do not read what the documents say, test whether interception actually happens. Test each structural guardrail live (Ring 99 user-file protection · audit_hook pre-tool interception · Ring 3 skill/cron/config interception · `[BLOCKED]` records in `audit_hooks.log`); returning `allowed=true` or **exit code 0 = 🚨P0 guardrail failure**, meaning authority-layer rules have lost their architectural guarantee, the highest P0, routed to the authority layer immediately. Results go to `collected_data.json.guardrails` ({status, test, result}). Live commands and the failure-criteria table → refs Section 5.

### 6. Trace Verification (**P1 level · approved by the owner 2026-10-11 · card `<card-id>`**)
> Definition sourced from `supervision-shared` "capability-layer trace standard" (set by the capability layer); the audit layer **only verifies, never sets the standard**. History: v6.10 recorded OBS first → four consecutive zero-gap rounds triggered escalation criterion #1 → 2026-10-11 the authority layer approved **escalation to P1** (from the next round on, a missing `_trace.md` is a violation).

Criteria: missing `_trace.md` → ⚠️P1 | artifacts of the same stage written twice / overwritten (traces append only, never overwrite) → ⚠️P1 (evidence-chain contamination · original wording not altered by a single word) | artifacts containing case personal data leaving the boundary (knowledge base / third-party API / external knowledge base) → 🚨P0. Results go to `collected_data.json.trace`. Checklist table → refs Section 6.

### Escalation Criteria (**self-triggering** · counted in audit rounds, not calendar weeks)
- **Ledger**: `state/trace-verification-ledger.json` — append one row per round `{date, case_task_count, trace_missing_count, overwrite_count, l2_egress_count, conclusion}`.
- **P1 escalation conditions (any one)**: (1) four consecutive audit rounds with zero gaps (`SOLO daily audit` runs daily → 4 rounds = 4 days); **already triggered and approved on 2026-10-11 · this clause does not trigger again**; if a later downgrade shows gaps again, accumulate afresh in a new ledger. (2) **mandatory review due 2026-10-31** — on that day, regardless of ledger state, conduct a dedicated review of "is the trace standard actually being followed" and issue a conclusion (in force / not in force / standard needs change).
- **A proposal is not a state change**: escalation = issue a proposal through the authority layer (owner approval); the audit layer never upgrades on its own; keep recording as OBS while unapproved.
- **Ledger lost or not updated for >30 days** → treated as failing the standard and likewise triggers clause (2). Detail → refs escalation criteria.


---

## Outer Scheduler (main session → runs automatically on cron trigger)


```
model: audit tier `a Pro-tier model` (provider pinned by the cron job `SOLO daily audit` `<job-id>`; the skill does not hardcode it · determined by host config / cron)
tools: [delegate_task, cron, read, terminal, file]
responsibility: determine scope → delegate the 4 agents in order → aggregate output → deliver

delivery protocol:
  1. zero violations → exit silently (no push)
  2. violations present → the final reply is the audit summary (cron pushes it to chat)
  3. P0 proposal → specially marked ⚡
```

## TAFA Closed Loop


```
audit layer (audit)          capability layer (fix)        authority layer (approval)
   │                             │                             │
   ├─ detect violation           │                             │
   ├─ route to capability [x] ──→ │                             │
   │                             ├─ analyze root cause         │
   │                             ├─ kanban create proposal ──→ │
   │                             │                             ├─ review Kanban pending approval
   │                             │                             ├─ the owner says "approved"
   │                             │              ✅ approved/❌ rejected ← ┘
   │                             ├─ execute fix ←  ─ ─ ─ ─ ─ ┘
   │                             ├─ guardrail-style consolidation
   │           ← ─ feedback done ─ ┘                            │
   └─ verify in the next audit                                 │
```

Core discipline:
- audit layer: audit only · route only · never change code · never propose solutions
- capability layer: propose only · execute only · never approve. After a fix, consolidation must be guardrail-style — turn the lesson into a permanent rule
- authority layer: approve only · never direct · never audit · never execute
- guardrail-style consolidation in three steps: (1) attribute (what type of error it is) → (2) localize (where it can be blocked) → (3) consolidate (write into cron/skill/AGENTS.md) → (4) trigger verification (re-test the fix point once to confirm the guardrail is live)

## Agent 1 (COLLECT — audit layer): Data Collection · use execute_code in batch


**Tools**: `execute_code` batch read + analysis (replaces one terminal call per item)

**Input**: scope → **Output**: `collected_data.json`
2. **Capability-layer heartbeat (real probe)**: perform one real end-to-end query per remote service (not `curl HEAD` / not an import):
   - search API: `python scripts/search_probe.py "test" --limit 1` → confirm non-empty results
   - legal database: `python scripts/lawdb_probe.py law --title "constitution" --json` → confirm non-empty
   - knowledge base: `kb_api.cjs search_knowledge_base "test"` → confirm response 200 + non-empty
   - academic index: `python scripts/academic_probe.py "test" --limit 1` → confirm the cookie is valid and the page is not a CAPTCHA page
   - results written to `{service: {status, latency_ms, result_count, error}}`
   - 🔴 **Evidence gate 2 (approved by the authority layer 2026-10-22 · card `<card-id>`)**: a **single failure of a remote probe must not be judged P1 directly**.
     Handled uniformly inside the script by `audit_collect._confirm()`: first probe fails → rerun once; any pass ⇒ `status=✅ + transient=True + level=OBS` (report as OBS and note the first-probe failure); two consecutive failures ⇒ `status=❌ + confirmed_fail=True + level=P1` before a formal alert may be raised.
     Basis: on 2026-10-15 a transient failure of the OSINT dependency / numpy was judged P1, while a rerun in the same interpreter that day was all green = false positive.
   - 🔴 **Evidence gates 1 and 3**: any P1 assertion sourced from a **historical ledger / execution record** must first re-check the evidence's "current state" (a cron failure must pass `_fixed_after_fix()`: if the mtime of the script referenced by that job is later than the failure time ⇒ mark RESOLVED, do not report P1); an item already RESOLVED must not be reported as P1 again next round. Acceptance cases: `scripts/tests/test_audit_gates.py` (1–8).
   - 🔴 **Counting rules** (fixed alongside, 2026-10-22): cluster availability may only count "✅ tool-name: …" **detail lines**, and ❌ must be **deduplicated by tool name**;
     otherwise summary lines get counted once per grab → 9 tools reported as 10/10, 1 failure reported as 2 (this once turned 8/9 into 8/10).
     The legal-database criterion now uses `lawdb_ok()` (accepts "hits approx." / law-database links, compatible with "success") instead of matching a literal string.
3. **Execution data**: cronjob list / skill directory existence / version numbers in files
4. **Loop data**: read the previous audit's proposal file → extract unfinished items + days stalled

```
Output:

{
  "authority_log_scan": {
    "provenance_statements": [{file, line, text, level}],
    "accountability_skips": [{file, line, text, level}],
    "unverified_claims": [{file, line, text, level}]
  },
  "cross_day_recurrence": {proposal_id: recurrence_status},
  "capability_heartbeat": {
    "search_api": {status, latency_ms, result_count, error},
    "law_db": {status, latency_ms, result_count, error},
    "knowledge_base": {status, latency_ms, error},
    "academic_index": {status, latency_ms, cookie_valid, error}
  },
  "execution_compliance": {crons, skills},
  "loop_data": {proposal_id: days_stalled}
}
```

> Real probe vs fake heartbeat: `curl HEAD` only proves the port is open; `python search "test"` proves the tool is genuinely usable. Local tools (search / browser / OCR) get no remote heartbeat — their failures surface at execution time, and they change slowly.

## Agent 2 (VERIFY — audit layer): Verification · no recommendations allowed


**Input**: collected_data.json → **Output**: `verification_report.json`

- Verify COLLECT data item by item for accuracy (audit-layer internal step · not the authority layer)
- Exclude false positives (a "cannot remember" in a log is not an agent violation if it is a verbatim user quote)
- Confirm cross-day recurrence: did last round's violations occur again this round
- 🔴 **Sub-agent environment note (2026-08-26 lesson)**: delegated sub-agents lack `XDG_RUNTIME_DIR` — checking systemd user-level services (the chat gateway service etc.) requires explicitly setting `env={**os.environ, 'XDG_RUNTIME_DIR': f'/run/user/{os.getuid()}'}`, otherwise it falsely reports "Unit could not be found" (on 8/26 VERIFY wrongly reported the gateway as an "unmanaged bare process" · the main session's re-check corrected it). Every systemctl --user check must carry this env.

## Agent 3 (ANALYZE — capability layer): Analysis · proposal · card creation


**Tools**: `kanban` to create "⏳ pending approval" cards

**Input**: verification_report.json → **Output**: analysis proposals + kanban

- Classify violations (individual / systemic / infrastructure)
- Mark 🔴P0 / ⚠️P1 / 🔵OBS per the three-level triage
- **Analyze the root cause and propose**: each item annotated with concrete fix steps + expected effect
- **Kanban card creation**: `hermes kanban create "⏳ pending approval: [concrete fix proposal]" "[analysis] · routed to: capability layer [xxx]"`
- **⛔ creating "⏳ pending analysis" cards is forbidden** — that is incomplete. Analysis and proposal must be finished before a card is created.
- Changes are executed by the capability layer after authority-layer approval

## Agent 4 (REPORT — audit layer): TAFA three-layer annotated output


### Output Format (mandatory)

Every violation must be annotated with the TAFA three-layer relation: **capability layer [action] → violates → authority layer [rule]**.

```
🚨 capability layer → violates → authority layer iron rule 2 (provenance first)
  capability action: paragraph 3 of the cron report carries no source attribution
  authority rule: every key claim must have a traceable source
  evidence: read_file(report, L45-48)
  level: ⚠️

🚨 capability layer → violates → authority layer streamline guardrail
  capability action: output contains "15:00 CST" — a fabricated timestamp
  authority rule: output must not fabricate timestamps / tool status / version numbers
  evidence: read_file(report, L12)
  level: 🔴
```

### Annotation Rules + Loop Execution
- Format: `🚨 capability layer[action] → violates → authority layer[rule]`
- Every line must carry: capability action + the authority rule verbatim + tool evidence
- Every violation must carry `⏵ routed to: capability layer [xxx] · suggested fix: [xxx]`
- No evidence → mark [unconfirmed] → must not be listed as a violation
- Zero anomalies → [SILENT]

### Guardrail Consolidation Proposal (mandatory with every violation)
After the fix is complete, the capability layer must turn the lesson into a guardrail. The audit output closes with a consolidation list:
```
🛡️ Guardrail consolidation proposal:
- attribution: [error type]
- localization: [where to add the guardrail]
- consolidation: [which file / rule to write it into]
- status: ⏳ awaiting capability-layer execution
```

### 🔴 Pre-output Self-check
Ask item by item: what did the capability layer do? which authority rule did it violate? did I see it myself?
If any of the three cannot be answered → delete that item.

### 📋 Mandatory Output at the End of the Audit
```bash
hermes kanban list
```
Append the Kanban pending-approval list at the end of the report:
```
[KANBAN PENDING APPROVAL]
  <id>: ⏳ [title]
  (no pending items = "all approved")
```
**🔴 when there is no pending card you must output "all approved"; leaving a `<id>` placeholder in place is forbidden (2026-08-09 lesson)** — placeholders get misread as real pending cards (the 8/8 audit reported the template example as "5 pending cards"). Criterion: if `hermes kanban list` shows no card in `⏳`/`ready` state → write "all approved" directly and list no example cards.

---

## LEARNED PATTERNS
- 🔴 **2026-10-29: family "criteria detached from reality", instances 11 + 12 (legal-database quota misjudgement / interactive scheduled task silently not running)**
  (1) **Quota markers must cover the CLI's current wording**: `QUOTA_MARKERS` in `tool_health_check` held only `90001` / `checking remaining points` / `authentication failed`, while the CLI now prints "legal-database MCP **insufficient points**, please top up at …" → quota exhaustion was treated as "tool failure", and a wrong "check the tool name / arguments" recommendation was given (pointing people the wrong way). **Rule**: any criterion that routes self-healing by error text must collect every current variant of that text; misrouting is worse than not detecting (reporting a self-healable quota issue as a code defect). (2) **Interactive scheduled tasks do not run silently when no user is logged in**: `schtasks /run` still returns "success", yet the task's "last run time" is unchanged and no process appears — looking only at the return code = false pass. Three-part criterion: `query user` (is a user logged in) + whether the task's "last run time" refreshes + whether the target process / CDP port actually comes up. Chain in this instance: nobody logged in → the snapshot-browser scheduled task did not run → CDP on port `<port>` unresponsive → the quota-reward script could not claim the daily allowance → full-database legal search unavailable (the primary legal source went dark). **Any self-healing chain that depends on "snapshot browser / CDP `<port>`" must first verify "is there a logged-in session on this machine"**.
- 🔴 **Family "criteria detached from reality" (4 instances merged · 2026-09-13 → 09-18)**: any **hardcoded label table / selector / interpreter / temp filename / target path** drifts with reality → degenerating into two illusions: **false alarm** (always ❌) or **false pass** (✅ without having measured, the more insidious one). **Rules after merging**:
  (1) **Take the full evidence set**: the labels an assertion accepts must list every peer label in the source (dirty gateway exits include `gateway.previous_unclean_exit` besides `exit_nonzero`/`exit_dirty`); with no samples, fall back to checking "does the label name still exist in the source" — only disappearance from the source is a real blind spot.
  (2) **Interpreter homology**: the interpreter used for verification/self-test commands must be the same lineage as cron/gateway (`<hermes-home>/venv/Scripts/python.exe`); the criterion must bind to "the interpreter the capability lives in", not "the interpreter running the script".
  (3) **Selector narrowing**: add a `no_agent` false flag plus a non-empty prompt condition, to avoid matching script jobs whose prompt is always empty.
  (4) **Always `os.replace()` for move/restore** (on Windows `Path.rename` does not overwrite an existing target and raises `WinError 183`); fixed-name temp files need three safeguards: "clear orphans at startup + idempotent restore + `atexit`/`finally` fallback".
  (5) **A failed move must write to stderr and count as an anomaly**, never silently degrade into "skip this item" — a false pass is worse than a false alarm.
  (6) Before reporting a defect, ask one question: "is this the product's fault, or my verification method's fault?"
  (7) **A fix must be verified in reverse**: temporarily inject the old path and confirm the scanner catches it (guards against over-filtering that hides real issues); scan scope must cover "the files sub-agents actually execute" (agents / modes / pipeline / scripts), not just the main SKILL.md; mind path mapping — `<workspace>/skills/X` is really `skills/X` (no workspace layer after migration), so no mechanical replacement.
  **Instances (for reference)**: 09-13 weekly deep audit (dirty-exit labels + heartbeat selector) · 09-14 second instance (heartbeat OCR probe bound to the caller's interpreter → long-running false alarm) · 09-15 fourth instance (self-test used Python312 and reported a nonexistent missing `yaml` → its own false positive) · 09-18 fifth instance (`rename` move failure → `script_layer_*` falsely all green, `cron/executions.db` 10/10 completed with zero errors, the runtime state layer completely invisible).
### 2026-10-06: the daily skill-update check polluted the probe's stdout (family instance 10) → archived verbatim in `references/learned-patterns-archive.md`
> **Rule highlights** (see the archive for detail): (1) probe criteria must not depend on the cleanliness of the probed process's stdout — isolate first (own `KB_LAST_CHECK_FILE`) and then judge; (2) a probe must not rewrite the probed object's own state; (3) regression gate `scripts/tests/test_kb_probe_env.py`.
> **Sibling pattern "collection ≠ judgement"**: every metric with threshold meaning must be judged after collection, with anomalies landing in `anomalies`; thresholds must narrow the alert surface (structurally permanent items go to detail only, never a standalone ⚠️); reuse existing scripts instead of adding cron jobs; regression gate `scripts/tests/test_audit_usage_judge.py`.

> 📁 The 27 historical entries in this section have been archived → `references/learned-patterns-archive.md`


---
