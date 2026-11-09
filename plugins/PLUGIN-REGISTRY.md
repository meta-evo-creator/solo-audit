# PLUGIN-REGISTRY.md — Skill Index

> SOLO Fleet Registry — an index of host runtime skills.
> Skills live under `skills/`, and the host runtime framework discovers and activates them automatically from the `description` keywords in SKILL.md.
> For the three-axis classification system see [`FLEET-CLUSTER.md`](../FLEET-CLUSTER.md)
> Last updated: 2026-06-16 (three-axis cluster classification added)

---

## Framework Plugins (host runtime framework level)

| Plugin | Purpose |
|:-------|:--------|
| memory-core | Memory management (short-term + long-term + dreaming) |
| memory-wiki | Compiled knowledge-base wiki |
| model-adapter | Model adapter for a Pro-tier model |
| enterprise-chat-connector | Enterprise messaging channel |
| chatbot-connector | Chatbot channel |
| stealth-browser-engine | Anti-detection browser engine |

## Scene (scene activation)

| Scene | priority | keywords | task_types |
|:------|:--------:|:---------|:-----------|
| deep-research | 3 | deep research, technology assessment, industry analysis, multi-source cross-check | L3 |

## host runtime Skills (automatic skill discovery)

| Skill | emoji | Cluster | Type | Ring | Trigger words | Architecture |
|:------|:-----:|:--------|:-----|:----:|:--------------|:-------------|
| solo | ⛳ | ⚙️ tools | meta-skill | 2 | fleet inspection, system architecture, separation of powers, kernel audit | separation of powers v2.1 (kernel + audit + skills) |
| solo-audit | 🔍 | 🔒 audit | meta-skill | 3 | daily audit, file-structure audit, iron-law benchmark, root-cause analysis | solo-audit v5.0.0 (3-agent pipeline, chained provenance, heterogeneous verification) ⛔ restricted |
| discipline-inspection | ⚔️ | 🏛️ oversight | pipeline orchestration | 3 | discipline review, case characterization, sanction recommendation, interview outline, complaint verification, enforcement review, violation finding, attribution analysis | 8-agent pipeline (Scope→Search→Audit→Analyze→Draft→Review→Revise→Publish) v1.0.0 ⛔ restricted |
| supervision-inspection | 🔍 | 🏛️ oversight | pipeline orchestration | 3 | inspection and patrol, political health check, inspection report, issue working paper, four implementations, inspection rectification, top-leader oversight | 8-agent pipeline (Scope→Search→Audit→Analyze→Draft→Review→Revise→Publish) v1.1.0 ⛔ restricted |
| hospital-inspection | 🏥 | 🏛️ oversight | pipeline orchestration | 3 | hospital inspection, large-hospital inspection, inspection report, issue rectification, party-building inspection, professional-conduct inspection, operations-management inspection, inspection feedback, inspection working paper, hospital self-check | 8-agent pipeline (Scope→Search→Audit→Analyze→Draft→Review→Revise→Publish) v1.0.0 ⛔ restricted |
| compliance-analysis | 🏛️ | 🏛️ oversight | pipeline orchestration | 3 | compliance analysis, procurement compliance, bidding compliance, research compliance, data compliance, health-insurance compliance, personal-information protection, compliance assessment, compliance advisory, risk rating | 8-agent pipeline (Scope→Search→Audit→Analyze→Draft→Review→Revise→Publish) v1.0.0 ⛔ restricted |
| babata-browser | 🦞 | 🦞 collection | tool wrapper | 1 | academic search, government sites, JS rendering, screenshots, web_fetch failure | stealth browser engine v3.2.0 |
| babata-superocr | 📝 | 🦞 collection | tool wrapper | 1 | image OCR, chat screenshots, screenshot text, scanned PDFs, handwriting recognition | dual OCR engine v2.0.0 (PDF + screenshots + batch) |
| solo-file-transfer | 📤 | ⚙️ tools | tool wrapper | 2 | Word to Markdown, upload to knowledge base, file upload to knowledge base | docx_to_md + ima_upload.cjs v1.1.0 |
| party-review | ⚖️ | 🔬 research | pipeline orchestration | 2 | party-building review, party-building good practices, party-building selection, four-dimension scoring, one-line commentary | seven-stage review (Validate→Parse→Score→Recommend→Draft→Review→Deliver) v1.1 ⛔ restricted |
| msf | ⚒️ | 🔬 research | pipeline orchestration | 2 | medical research, systematic review, evidence quality, group standard, technology roadmap, academic society | OPL stage-led v4.2.3 |
| deep-research | 🔍 | 🔬 research | pipeline orchestration | 2 | deep research, technology assessment, industry analysis, multi-source cross-check, L3 tasks | 8-agent pipeline (Scout→Merge→Analyze→Draft→Review→Revise→Publish) v4.0.0 |
| babata-search | 🌐 | 🦞 collection | tool wrapper | 0 | Chinese search, domestic engines, portal search, site search, domestic information retrieval | dual-engine browser automation v2.0 |
| officecli-xlsx | 📊 | ⚙️ tools | tool wrapper | 1 | .xlsx spreadsheets, Excel data processing, reports, data analysis | single-binary Rust + OpenXML v1.0.116 |

## Disabled / pending installation

| Plugin | Description | Status |
|:-------|:------------|:------:|
| blogwatcher | Blog monitoring (RSS/Web) | Installed but not fully configured |
