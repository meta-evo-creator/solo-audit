# Plugin Dispatch System
> SOLO v1.0 — kernel-level precise targeting + automatic legal-and-discipline escalation
> Position: Suit layer, Step 0.5 (after pre-checks, before the five-layer execution)

## Kernel-Level Rules (cannot be skipped)

**Automatic legal-and-discipline escalation**: if a task contains any legal-and-discipline keyword, its tier is at least L2. This rule runs before Dispatch.

Legal-and-discipline keyword set: `{discipline inspection, supervision, discipline, legal affairs, regulations, sanctions, accountability, inspection, patrol, party discipline, administrative discipline, Eight-Point Regulation, four malfeasances, triple major and one large}`

---

## How It Works

When a task enters the Suit layer, the kernel walks the `trigger_fingerprint` of every non-dormant plugin and matches on four dimensions — keywords + anti-keywords + task tier + priority — then outputs an activation list.

## Dispatch Flow

```
Task input
  │
  ├─ Step 0: Kernel Boot (native host runtime session startup)
  │
  ├─ Step 0.5: Plugin Dispatch
  │     │
  │     ├─ 1. Scan the trigger_fingerprint of all active/ + scene/ plugins
  │     ├─ 2. Four-dimension match per plugin:
  │     │     [keywords ∩ task terms] ≠ ∅  AND
  │     │     [anti_keywords ∩ task terms] = ∅  AND
  │     │     [task tier ∈ task_types]          AND
  │     │     [no higher-priority plugin with the same function conflicts]
  │     ├─ 3. Sort candidate plugins by priority
  │     ├─ 4. Output the activation list
  │     └─ 5. Label explicitly:
  │         🔌 Dispatch: med-research(p5) + stage-checkpoint(p8) + deep-research(p3)
  │
  └─ Step 1-5: Normal five-layer execution
```

## Matching Rules

| Rule | Description |
|:-----|:------------|
| **Hit wins** | If any keyword hits the task's core description, the plugin becomes a candidate |
| **Anti-exclusion** | If any anti_keyword appears, the plugin is excluded immediately |
| **Tier restriction** | A plugin activates only within the task_types it declares |
| **Priority arbitration** | When two plugins of the same kind both match, the higher priority wins |
| **Forced activation** | stage-checkpoint has priority=10 for L2/L3/Cron and cannot be overridden |

## Chain Mechanism (new in v7.1)

If a plugin's output JSON contains a `trigger_next` field, the kernel dispatches the next plugin automatically.

```
Plugin A output: { result, quality, trigger_next: "policy analysis" }
  → kernel Gate checks that quality passes → Dispatch policy analysis
```

Typical chain: deep-research → policy assessment (policy evaluation)
Orchestrator: policy-eval plugin (L3/priority 4)

## Fingerprint Format

Each `.plugin.md` file gains a `trigger_fingerprint` block at the top:

```yaml
# trigger_fingerprint:
#   keywords: []
#   anti_keywords: []
#   patterns: []
#   task_types: []
#   priority: 1-10
#   force_activate: true/false
```

| Field | Description |
|:------|:------------|
| keywords | Keywords/phrases that trigger the plugin |
| anti_keywords | Exclusion words — the plugin stays inactive if the task contains them |
| patterns | Scene patterns — more precise matching than keywords |
| task_types | Allowed MEV tiers: [L1, L2, L3] |
| priority | Priority 1-10 (10 = system-level forced) |
| force_activate | Whether to ignore all matching rules and activate regardless |

## Relationship to Human Judgment

Dispatch does not replace human judgment — it **narrows the candidate set**. The final activation list is a recommendation; a human may add or remove entries manually. Format:

```
🔌 Dispatch: med-research(p5) + stage-checkpoint(p8)
📋 Candidate but not activated: deep-research(p3) — overlaps with med-research
❌ Excluded: compliance-research(no keyword hit) | bias-check(priority below med-research)
✋ User can: reply "+deep-research" or "-med-research" to adjust manually
```
