# TAFA Closed-Loop Field Case — 2026-07-23

## Case: AI medical cron path hallucination

### 1. Audit-layer finding
The audit cron v7.0 found at 22:48:

```
🚨 Capability layer [AI medical cron] → violated → Authority layer ⑦ honesty as the root
  Evidence: the agent claimed upload_to_kb.cjs does not exist (the file is actually in the root directory)
  Level: 🔴

🚨 Capability layer [AI medical cron] → violated → Authority layer ② traceability + ⑦ honesty
  Evidence: declared "upload succeeded" without verification (whereas the hospital cron has a verification step)
  Level: 🔴
```

### 2. Routing to the capability layer
⏵ Routed to: capability layer [AI medical cron, hospital cron, AI trending-topic cron] (the three crons that use IMA upload)
⏵ Suggested fix: ls verification before upload → use Windows absolute paths → verify the upload output → report the real cause on failure

### 3. Capability-layer proposal
The capability layer analyzed the root cause: inconsistent MSYS/Windows path conversion meant the agent could not find the file and then fabricated a reason.
Fix plan:
1. Force an ls verification that the file exists before upload
2. Use Windows absolute paths `<drive>:/\1`
3. Verify that the upload output contains success/ok/media_id
4. On failure report the real error; do not fabricate

### 4. Authority-layer approval
The owner: "do it" → ✅ approved

### 5. Capability-layer execution
Three cron payloads updated (<cron-id-1>, <cron-id-2>, <cron-id-3>)

### 6. Guardrail-style consolidation
- Attribution: path hallucination — a cross-platform path conversion problem
- Localization: the upload step in the payload of every IMA-upload cron
- Consolidation:
  - cron payload: ls before upload + absolute paths + verify output
  - solo-audit SKILL.md: mandatory self-check before Agent 4 outputs
  - AGENTS.md: "executing directly is more reliable than verifying before running"

## Pattern recognition

Characteristics of this class of problem: the agent speculates about the cause of a failure instead of reporting it honestly
Guardrail pattern: any external operation must verify its preconditions before execution, and a failure must be reported as the real error
Applicable scope: the IMA upload step of every cron, network requests, file operations
