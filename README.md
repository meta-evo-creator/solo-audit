# SOLO Audit — a TAFA-native self-audit loop for AI agent fleets

An audit agent that checks whether a working AI-agent fleet obeys the rules its human owner set,
routes violations back to the layer that can fix them, and verifies that the guardrails it relies on
actually block things — instead of trusting documentation that says they do.

This repository is the English edition of the audit layer and of the four specialist agents it
supervises. The Chinese originals — including the statutory texts, which are not translated because a
translated statute is a commentary rather than the statute — are archived under `legacy/zh/`.

## Why this exists

Most "agent governance" is a policy document. Policies do not stop an agent from deleting a file.
What stops it is a structural interceptor that returns non-zero on the attempt. This project is built
on one test:

> A rule is real only if a violating action is actually blocked. If the block does not fire, the rule
> is decoration, and that is the highest-severity finding the audit can produce.

## Three layers

| Layer | Holds | Does |
|---|---|---|
| **Authority** | A human owner, personally. Seven iron rules, five operating principles, a five-stage reasoning loop. | Approves, rejects, changes the standard. Nothing else. |
| **Capability** | Skills, sub-agent pipelines, scheduled jobs. | Proposes, executes, hardens lessons into guardrails. |
| **Audit** | This agent. | Finds violations, assigns severity, routes them. Never fixes, never proposes fixes. |

The separation is the point: the auditor that also repairs has an incentive to grade its own work.

## What gets audited

1. **Iron rules** — sourcing discipline, authority boundaries, honest degradation reporting, injected-instruction resistance, no writes into user-owned file areas, no fabrication.
2. **Operating principles** — inflation trap (check whether an existing mechanism failed before adding a rule), parsimony, priority, entropy reduction, guardrails-over-goodwill.
3. **Reasoning loop** — whether work actually went through the five stages, or skipped them and called the result analysis.
4. **Agent performance** — observation only. A slow or hallucinating agent is not a rule violation; it is a datum.
5. **Structural guardrails** — each interceptor is exercised for real. A blocked operation that returns success is a P0.
6. **Trace completeness** — every case-type task must leave an append-only trace file; missing trace is P1, silent overwrite is P1, sensitive-data egress is P0.

## Severity and routing

| Level | Meaning | Route |
|---|---|---|
| P0 | Privacy breach, authority breach, data loss, production impact, or a guardrail that does not fire | Human approval, immediately |
| P1 | Path, format, verification or threshold defects | Capability layer proposes; executes after approval |
| OBS | Trend, performance drift, non-urgent | Recorded only |

Zero findings produces no output at all. A daily audit that always has something to say trains its
reader to stop reading.

## Four-agent pipeline

```
COLLECT  →  VERIFY  →  ANALYZE  →  REPORT
(audit)     (audit)    (capability)  (audit)
```

- **COLLECT** gathers evidence: rule-pattern scans, real end-to-end probes of remote services, scheduled-job state, open proposals.
- **VERIFY** removes false positives, checks cross-day recurrence, and re-confirms anything inherited from a previous ledger before it is reported again.
- **ANALYZE** classifies root cause and files a proposal card. It never files a "to be analysed" card — an unanalysed card is not a deliverable.
- **REPORT** emits findings in a fixed three-layer form: `capability[action] → violates → authority[rule]`, each with tool evidence and a routing line.

## Evidence gates

Two failure modes are cheap to prevent and expensive to miss: **false alarms** and **false passes**.

- A single failed probe is not a finding. Fail once, retry once; transient failure is recorded as OBS with the first-failure note. Only two consecutive failures escalate.
- A previously reported item whose referenced artifact was modified after the failure is marked resolved and not re-reported.
- Availability counts are derived from detail lines, deduplicated by tool name — never from summary lines, which double-count.
- Anything taken from a stored ledger is re-checked against the current state before it becomes a P1.

## The recurring lesson: criteria drift away from reality

Twelve recorded incidents share one shape: a hard-coded label, selector, interpreter, temporary
filename or target path silently stops matching the world, and the check degrades into either a
permanent false alarm or — worse — a permanent false pass. The rules distilled from them:

1. Collect the complete label set from the source, not the one label you saw.
2. Verify with the same interpreter that runs the job; a check bound to the wrong interpreter tests nothing.
3. Qualify selectors so empty-prompt script jobs cannot match.
4. Move and restore files with an atomic replace, never a rename that fails on an existing target.
5. A failed move is an error, not a skip. Silent degradation is worse than a noisy alarm.
6. Before reporting a defect, ask: is this the product's fault, or my verification method's?
7. After a fix, inject the old path back and confirm the scanner still catches it.

**Anti-gaming rule.** Thresholds and criteria belong to the authority layer. When an alert is
uncomfortable, the fix is the system's state, not the ruler. Changing the measurement so the warning
disappears is itself a violation.

## Four specialist agents

The audit layer also supervises a four-node case pipeline: **CT** (clue triage) → **IO** (interview
preparation) → **CD** (characterisation and sanction tiering) → **PD** (disposition proposal drafting).
See [docs/four-specialists-en.md](docs/four-specialists-en.md).

Each specialist is bounded by a no-authority contract: it produces an analytical candidate, never a
decision. Identity, case data and regulations stay local; no case is ever sent to a third-party
service.

## Layout

```
README.md                       this file
SKILL.md                        audit skill, English, v6.13.0
AUDIT_LEARNINGS.md              incident log and the rules distilled from it
docs/                           full specification, skill edition, specialist pipeline
references/                     criteria, metrics, guardrails, incident patterns
plugins/                        plugin registry and dispatch contract
artifacts/                      system overview
memory/                         standing knowledge compilation
skills/solo/                    governance skill package (authority / capability / audit)
skills/solo-audit/              audit skill, English edition
scripts/                        collection and dispatch helpers
legacy/zh/                      Chinese originals and untranslated subject-matter packages
```

### Versions

| Document | Version | Status |
|---|---|---|
| `SKILL.md`, `docs/solo-audit-v6.13.0-en.md`, `skills/solo-audit/SKILL.en.md` | 6.13.0 | **current** |
| `legacy/zh/SKILL.md` | 6.9 | superseded, archived |
| `legacy/zh/skills/solo-audit/SKILL.md` | 5.0.0 | first snapshot, archived |

Everything in the main tree is English and current. `legacy/zh/` is a record, not a reference.

## Scope of publication

These documents describe method, not clients. Case material, organisational identity, personnel,
credentials and internal paths are excluded by construction — anything that would identify a matter
or a person does not belong in a public repository, and is not here.
