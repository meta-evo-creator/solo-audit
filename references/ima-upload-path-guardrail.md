# IMA Upload Path Hallucination · A Consolidated Guardrail

> Found: 2026-07-23 audit SP-030 | Fixed: closed the same day

## Problem

While performing an IMA upload, the capability-layer cron claimed that `upload_to_kb.cjs` did not exist (the file did in fact exist).
Root cause: the MSYS path format (`/c/<path>`) did not match the Windows path Node.js expects (`<drive>:/\1`).
The agent reported "the script does not exist" without verifying — a violation of honesty as the root.

## Guardrail (consolidated into three cron payloads)

```
Pre-upload verification steps:
1. ls "<repo>/skills/ima-skill/upload_to_kb.cjs" — confirm the file exists
2. cd "<repo>/skills/ima-skill"
3. node upload_to_kb.cjs "KB_ID" "" "<repo>/memory/..." "title" 2>&1
4. Verify the output contains "success" or "ok" or media_id → report ✅
5. On failure → report the real error cause, do not fabricate
```

## General principles

A script path already confirmed through deployment → execute directly + report errors on failure.
Do not verify with search_files → inconsistent path formats produce false negatives.
Node.js requires Windows absolute paths (`<drive>:/\1`, not `/c/...`).
