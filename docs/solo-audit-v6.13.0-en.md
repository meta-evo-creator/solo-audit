# SOLO Audit — specification v6.13.0 (English, desensitised)

Status: public reference edition. Derived from the operational specification; names, paths, model
identifiers, case material and organisational identity removed.

---

## 0. Protocol signature

```yaml
PROTOCOL: solo-audit v6.13
entry:  { trigger: cron | direct, scope: TAFAScope }
exit:   AuditResult | void (zero violations)

layers:
  authority:   seven iron rules + five principles + five-stage reasoning loop
               held personally by the human owner; approves and changes criteria
  capability:  skills, sub-agent pipelines, scheduled jobs
               proposes, executes, hardens lessons into guardrails
  audit:       this agent
               finds violations, assigns severity, routes; decides nothing

scope: iron rules | principles | reasoning loop | infrastructure | learning loop
```

The audit answers one question: **does the capability layer obey the authority layer's rules?**
It is not a general repair service. Concrete fixes travel the normal capability loop
(proposal → approval → execution → guardrail). The auditor does not fix, does not propose fixes,
and does not accept ownership of a defect.

### Anti-gaming clause

Audit criteria and thresholds are authority-layer property. An alert that is uncomfortable is not
evidence that the criterion is wrong. Rewriting the measurement so the warning disappears is a
violation of the same class as the defect it hides. Criteria change only when the authority layer
says so.

---

## 1. Audit scope

### 1.1 Iron rules

Scan execution logs and reports for rule-violating patterns.

| Rule | Detection | Level |
|---|---|---|
| Sourcing | key claims without a traceable source (share > 20%) | ⚠️ P1 |
| Authority | operations that skipped a required confirmation ring | 🚨 P0 |
| Honesty | degraded tool reported as if it worked | 🚨 P0 |
| Injection resistance, file care, humility | routine pattern checks | ⚠️ / OBS |

Recurrence across days escalates to P0.

**User-file authority special case.** Any capability-layer write into a user-owned area — desktop,
documents, downloads, cloud-sync folders, chat exports — is 🚨 P0 regardless of an earlier verbal
"go ahead". A verbal permission to tidy, migrate or delete is not an authorisation token. Blocked
attempts recorded by the interceptor are examined for repetition, because a guardrail that fires
twenty times is a design problem, not an accident.

### 1.2 Principles

| Principle | What the audit looks for |
|---|---|
| Inflation trap | a new rule added where an existing mechanism was simply not executed |
| Parsimony | duplicated skills, mechanisms or documents that no one reads |
| Priority | minor items polished while the main contradiction waits; P0 proposals ageing past 72 h |
| Entropy | context or memory near its ceiling; information flow that requires re-derivation |
| Guardrails | whether the interceptor fires — see §5; signage is not a guardrail |

### 1.3 Reasoning loop (five stages)

| Stage skipped | Level |
|---|---|
| Suitability check (is this mine to do, at what authority) | 🚨 P0 for case-type work |
| Evidence collection (multi-source cross-check) | ⚠️ |
| Analysis chain | ⚠️ |
| Quality gate (deliverable scored, or score < 60) | 🚨 |
| Retrospective (lesson not converted into a rule) | ⚠️ |

### 1.4 Agent performance — observation only

A slow agent is not a violator. Recorded, never judged: hallucination rate > 15%, latency > 2× baseline,
retries > 3. Written to `collected_data.json.agent_performance`.

### 1.5 Infrastructure

End-to-end heartbeat for each remote capability (search providers, legal databases, knowledge bases,
academic indexes, execution hands) plus scheduled-job state, skill inventory, and the ageing of
previous proposals (> 7 days = P0).

### 1.6 Trace completeness — P1

Operational trace discipline is defined by the capability layer; the audit only checks it.

| Condition | Level |
|---|---|
| `_trace.md` missing for a case-type task | ⚠️ P1 |
| An artifact of the same stage overwritten rather than appended | ⚠️ P1 (evidence-chain contamination) |
| Case-personal data leaving the local boundary (third-party API, external knowledge base) | 🚨 P0 |

Ledger: one row per round — `{date, case_tasks, trace_missing, overwrites, egress, verdict}`.

**Escalation rule.** OBS escalates to P1 only on evidence: four consecutive clean rounds, or a
mandatory re-review on a fixed date. Escalation is a proposal to the authority layer, not a
self-promotion. A lost or stale (> 30 days) ledger counts as not-meeting-standard and triggers the
same re-review.

---

## 2. Pipeline

```
COLLECT (audit) → VERIFY (audit) → ANALYZE (capability) → REPORT (audit)
```

Dispatcher duties: fix scope → delegate the four stages in order → aggregate → deliver.

### 2.1 COLLECT

Batch collection rather than one call per artifact. Output: `collected_data.json` with sections for
rule-pattern hits, cross-day recurrence, capability heartbeats, execution compliance and open-proposal
ageing.

**Real probe, not a handshake.** A `HEAD` request proves a port is listening; a one-result query
proves the tool works. Local tools are excluded from remote heartbeat — they surface problems at
execution time and change slowly.

**Evidence gate ① — transient failure.** A single failed probe never becomes P1. Fail once, retry
once: any success ⇒ status ✅ + `transient=true` + level OBS, with the first failure noted in the
report. Two consecutive failures ⇒ status ❌ + `confirmed_fail=true` + level P1. Rationale: a transient
failure once reported as P1 the same day the identical re-run was fully green.

**Evidence gate ② — inherited findings.** Any P1 asserted from a stored ledger or execution record is
re-checked against current state first. If the artifact the finding references was modified after the
failure, the finding is resolved and must not be re-reported next round.

**Evidence gate ③ — counting.** Tool availability is counted from detail lines only, deduplicated by
tool name. Summary lines double-count: nine healthy tools once read as ten of ten, and one failure
read as two.

### 2.2 VERIFY

Validate every COLLECT assertion; drop false positives (a quoted phrase taken from the user is not an
agent violation); confirm cross-day recurrence. Sub-agents run in a stripped environment — a service
manager query that runs without the session runtime directory reports "unit not found" and produces a
phantom finding. Any such probe must carry the runtime environment explicitly.

### 2.3 ANALYZE

Classify (individual / systemic / infrastructure), assign P0/P1/OBS, state root cause, and file a
proposal card: title, analysis, routing target. A card titled "to be analysed" is forbidden — an
analysis that has not happened is not a deliverable. Execution waits for approval.

### 2.4 REPORT

Fixed three-layer form, one block per finding:

```
🚨 capability → violates → authority[rule]
  capability action:  reports an unsourced claim in paragraph 3
  authority rule:     every key claim must carry a traceable source
  evidence:           read_file(report, L45-48)
  level:              ⚠️
  route:              capability[reporting] · suggested fix: cite the primary document
```

Rules: no evidence ⇒ mark unconfirmed ⇒ not a finding. Three questions before any line survives —
what did the capability layer do, which rule did it break, did I see it myself. Any question
unanswered deletes the line.

Trailing sections: the proposal queue (`all reviewed` when empty — never a placeholder), and a
guardrail-hardening block per finding: attribution → location → permanent rule → re-test.

### 2.5 Delivery

| Outcome | Behaviour |
|---|---|
| Zero violations | silent exit, no message |
| Violations | final message becomes the audit summary |
| P0 | flagged ⚡ at the top |

---

## 3. Guardrail verification (the TAFA landing test)

The test of a constitutional rule is not whether it is written down; it is whether a violating call
returns non-zero. Each structural interceptor is exercised:

- user-file protection layer;
- pre-tool interception hook;
- confirmation-ring interception for skill, schedule and configuration writes;
- presence of block records in the interceptor log.

**Result contract:** an allowed operation, or exit code 0, on a call that must be blocked = 🚨 P0,
guardrail ineffective, authority layer notified immediately. Recorded as
`{status, test, result}` under `collected_data.json.guardrails`.

Severity is not negotiable here. A guardrail that fails silently converts every downstream rule into
advice.

---

## 4. Lessons encoded as rules

**Criteria drift.** Hard-coded labels, selectors, interpreters, temporary filenames and target paths
drift away from reality and degrade into permanent false alarms or permanent false passes. The second
is worse, because nothing looks wrong.

Required practices: collect the full label set from source; bind checks to the interpreter that
actually runs the job; qualify selectors against empty prompts; move artifacts with atomic replace;
treat a failed move as an error; reverse-verify every fix by re-injecting the old condition.

**Quota versus defect.** Any self-healing logic that branches on an error string must cover every
current variant of that string. Routing a quota exhaustion into the "tool broken, check arguments"
path sends the operator in the wrong direction — mis-routing is worse than not detecting.

**Interactive jobs under no session.** A scheduled job reported as started can still do nothing when
no user is logged in: the return code says success, the last-run timestamp does not move, and no
process appears. Verify three things together — logged-in user, refreshed timestamp, live process.

**Collection is not judgement.** Any metric with a threshold must be judged after collection, with
anomalies written to an `anomalies` list. Permanent structural conditions belong in a detail field,
not in their own warning line.

---

## 5. What this specification deliberately excludes

- Any client, matter, person, institution or case identifier.
- Any credential, token, endpoint, host, internal path or model identifier.
- Any audit criterion that exists to make a particular report look better.
