# SOLO Fleet Status

## Current Fleet (9 agents)

| Agent | Type | Status | Last Active | Health |
|:------|:----:|:------:|:----------|:------:|
| discipline-inspect ⚔️ | skill | 🟢 | today | v2.2.1 — 4-Agent独立子代理 + 任务分流规则 + 降级铁律 |
| solo-file-transfer 📤 | skill | 🟢 | today | docx→md + IMA上传 |
| MSF ⚒️ | skill | 🟢 | today | v2.0 + 2 report |
| babata-browser 🦞 | skill | 🟢 | today | CCDI验证 |
| babata-superocr 📝 | skill | 🟢 | 05-18 | dual-engine |
| tool-awakening | active | 🟢 | per-Suit | 门禁+预检+格式强制 |
| core-lessons | active | 🟢 | per-Suit | 26 lessons |
| cron-rules | active | 🟢 | 05-14 | cron安全 |
| deep-research 🔍 | skill+scene | 🟢 | today | v3.0 |

## Wiki Vault (115 pages)

```
discipline/
  ├─ 法规/        17   监察·党纪·政务·刑事·行政·关联法
  ├─ 指导性案例/  11   中纪委第一至第三批全覆盖
  └─ 方法论/       1   违规+有责·谈话四步法·11案对照
inspection/ (9子目录)
  01-基础制度 7 · 02-实务操作 13 · 03-协作机制 6
  04-自身建设 3 · 05-工作流程 2 · 06-模板工具 37
  07-政治理论 7 · 08-中大落实 4 · 09-医院制度 4
medical/        5   医师法·药品管理法·基本医疗法·药代管理·九项准则
(root)          3   索引+新旧对照
```

## Cron Tasks

| Task | Schedule | Next |
|:-----|:--------:|:----:|
| Memory Dreaming | 03:00 | ~3h |
| AI热点监测内参 | 04:00 | ~4h |
| 医院智慧监督内参 | 07:00 | ~7h |
| AI医疗案例监测 | 09:00 | ~9h |
| 法规月度更新检查 | 每月1日 | 6月1日 |

## Health

| Signal | Status |
|:------|:------:|
| Tool Failures | 🟢 0 |
| Cron Errors | 🟢 0 |
| Wiki Coverage | 🟢 115页 |
| Skill Drift | 🟢 aligned |

## Last Check

2026-05-21 16:00 CST | SOLO ⚡ — v2.2 + ripgrep 全面部署完成
