# Agent Reach Architecture Benchmark Analysis

> Source: research on Panniantong/Agent-Reach (59k stars, 4.7k forks, v1.5.0)
> Date: 2026-07-23
> Purpose: provide an external reference for the SOLO fleet's collection cluster and audit system

---

## Agent Reach core idea

**A capability layer, not yet another tool.** It does not read content — it handles selection, installation, health checks, and routing. Content reading is done by the agent calling upstream tools directly, with zero wrapper overhead.

---

## Three key design patterns

### 1. Channel + ordered routing across multiple backends

```
channels/twitter.py → twitter-cli ▸ OpenCLI ▸ bird
channels/bilibili.py → bili-cli ▸ OpenCLI ▸ search API (yt-dlp has been blocked and retired)
```

Each channel = an ordered list of primary + fallback. Changing the access method = reordering, no code rewrite. A broken backend degrades automatically — Bilibili blocked yt-dlp, so it switched to bili-cli, with zero user action.

**Implication for SOLO**: each search tool in the collection cluster currently has only a single path. A degradation chain can be defined:
- Chinese search: babata-search → web_search native
- International search: tavily-search → web_search native
- Regulation search: legal-db-search → web_search + direct scraping of the legal database official site

### 2. Real probing (end-to-end testing)

`agent-reach doctor` runs an end-to-end test against every backend — not `which twitter-cli`, but actually sending a request and validating the returned data. Broken ones get a repair prescription.

**Implication for SOLO**: the fleet health check currently only verifies directory existence + API HEAD reachability. It should be upgraded to: each collection skill performs one real query and validates that the returned data is valid. Already recorded in `fleet-health-check.md`.

### 3. Install-time registration (one click eliminates multi-table sync)

After installation it automatically writes SKILL.md into the agent's skills directory, so the agent knows which tool to call for the corresponding need going forward.

**Implication for SOLO**: our current three-table sync of SOUL.md → FLEET-CLUSTER.md → PLUGIN-REGISTRY.md is the root cause of the drift found in the 07-23 audit (5 skills missing). Consider converging registration information into a single table, PLUGIN-REGISTRY.md (already the host runtime native format).

---

## Heartbeat scope principle (established by this audit)

Only check remote services (an API may go down), do not check local libraries (once installed they do not suddenly disappear).

| Should check | Should not check | Reason |
|:-----|:------|:-----|
| Tavily API | easyocr import | Remote may go down; a local package never fails after installation |
| IMA API | paddleocr import | Same as above |
| legal database API | yt-dlp version | Same as above |
| CNKI cookie expiry | Python package version | Cookies expire; package version changes do not block |

---

## Benchmark summary table

| Dimension | Agent Reach | SOLO fleet | Gap |
|:-----|:-----------|:----------|:-----|
| Multi-backend routing | Primary + fallback list per channel | Single path per tool | No degradation chain |
| Health check | End-to-end real probing | Directory existence + API HEAD | Cannot tell whether a tool is actually usable |
| Skill registration | Install → SKILL.md → auto-discovery | Hand-maintained three registries | 5 omissions found on 07-23 |
| Failure recovery | Backend switch invisible to the user | Requires a human to notice, then fix manually | Passive |
| Zero-config out of the box | 6 channels with zero config | Each skill configured independently | High barrier for onboarding a new skill |
