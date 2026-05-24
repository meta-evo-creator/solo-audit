---
name: msf
version: 4.2.2
description: |
  医学社会组织发展研究工场 v4.2.2 — OPL stage-led（+ 反爬检测 + 二次Review + 评分标准化）。Stage as delivery unit + Stage Receipts + Attempt Ledger + Per-Stage Quality Gates + Review Ledger + Authority Boundary。设计来源：gaofeng21cn/one-person-lab(OPL)+med-autoscience(MAS)。
  Use when: 医学研究,系统评价,证据质量,团标,技术路线图,学会,干细胞,AI医疗
  NOT for: 基金申请,合规审查,纪检分析。
metadata:
  openclaw:
    emoji: ⚒️
    requires:
      sessions: sessions_spawn 可用
---

# MSF v4.2 ⚒️ — OPL 注入版

> 设计来源：gaofeng21cn/one-person-lab (OPL stage-led framework) + med-autoscience (MAS)
> **v4.2 核心升级：吸收 OPL 六大模式 —— Stage Receipts / Attempt Ledger / Per-Stage Gates / Review Ledger / Authority Boundary / Recovery**

## OPL 核心原则（已吸收）

| OPL 模式 | MSF 实现 |
|:---------|:---------|
| **Stage as delivery unit** | 每个 Agent 是独立 stage，有自己的 goal/input/output/gate/receipt |
| **Stage Receipts** | 每个 Agent 输出含 quality 自检 + handoff 签字 |
| **Attempt Ledger** | 记录每阶段重试次数、改进内容、历史分数 |
| **Per-Stage Quality Gates** | Scout→完整性检查 / Analyze→PRISMA完整 + A≥2 / Draft→引用全覆盖 / Review→独立审计 |
| **Review Ledger** | 独立于 Evidence Ledger 的审稿决策轨迹 |
| **Authority Boundary** | AI: Scout/Analyze/Draft/Revise | Human: 最终结论/医学判断/对外发布 |
| **Recovery** | 任一 stage FAIL → 标记原因 → 退回上一 stage 重试（保留 attempt 历史） |

## ⚡ Solo Status 协议（强制）

每次 Stage 切换时更新 `./solo/pipeline-status.json`，与 DR/DI 格式统一。

```json
{
  "pipeline_id": "MSF-20260524-xxx",
  "skill": "msf",
  "topic": "研究主题",
  "started_at": "ISO时间",
  "last_updated": "ISO时间",
  "phases": {
    "Stage 0: Contract": {"status": "completed", "detail": "stage_contract.yaml"},
    "Stage 1: Scout":    {"status": "running",   "detail": "4库搜索中"},
    "Stage 2: Analyze":  {"status": "pending",   "detail": ""},
    "Stage 3: Draft":    {"status": "pending",   "detail": ""},
    "Stage 4: Review":   {"status": "pending",   "detail": ""},
    "Stage 5: Revise":   {"status": "pending",   "detail": ""}
  },
  "running_subagents": []
}
```

## 5-Agent Stage 管线 (v4.2)

```
用户问题
  │
  ├─ Stage 0: 主会话 freeze Stage Contract (不可修改)
  │     → output: stage_contract.yaml
  │
  ├─ Stage 1: sessions_spawn Agent 1 (Scout) 
  │     → input: stage_contract
  │     → output: scout_results.json + receipt (completeness ≥ 80%)
  │     → gate: sources ≥ 10, databases ≥ 3, A-strong ≥ 2
  │
  ├─ Stage 2: sessions_spawn Agent 2 (Analyze)
  │     → input: scout_results + stage_contract
  │     → output: evidence_ledger.json + PRISMA_flow.json + receipt
  │     → gate: PRISMA 完整, A-strong ≥ 2, counter_evidence 已搜索
  │
  ├─ Stage 3: sessions_spawn Agent 3 (Draft)
  │     → input: evidence_ledger + PRISMA_flow + stage_contract
  │     → output: draft_report.md + receipt (citation coverage ≥ 95%)
  │     → gate: 每条 claim 有 source, 局限性已声明
  │
  ├─ Stage 4: sessions_spawn Agent 4 (Review) ⚠️ 独立审计, 硬门禁
  │     → input: 原始 contract + scout原始数据 + evidence_ledger + draft_report（只读）
  │     → output: review_ledger.json + quality_score + revision_plan
  │     → gate: score ≥ 80 → PASS | 60-79 → Stage 5 | <60 → REJECT/退回 Stage 3
  │     → ⛔ 不继承任何上游 Agent 上下文
  │
  ├─ Stage 5: sessions_spawn Agent 5 (Revise)
  │     → input: draft_report + review_ledger
  │     → output: final_report.md + revision_log.md
  │     → gate: P0 全修复, P1 ≥ 80%
  │     → 输出: original_review_score + revised_score_estimate
  │
  ├─ Stage 5.5: 主会话 Review 决策
  │     ├─ if original_review_score < 80 AND revised_score_estimate ≥ 80:
  │     │    → sessions_spawn Agent 4 (Review) 二次审计 final_report.md
  │     │    → 用新评分决定 PASS/REVISE/REJECT
  │     ├─ if original_review_score ≥ 80: → 直接进 Stage 6
  │     └─ 最多重试 2 轮 Revise→Review，超限→HUMAN_ESCALATION
  │
  └─ Stage 6: 主会话 Deliver
        → gate: review_score ≥ 80, P0=0, revision_applied ≥ 80%, 或 human_override
        → 检查 authority_boundary: 最终结论标记 ⏸️ HUMAN_APPROVAL
```

## Stage Receipt 格式（每个 Agent 必须输出）

```json
{
  "stage": "scout/analyze/draft/review/revise",
  "receipt_id": "MSF-YYYYMMDD-{stage}-001",
  "attempt_number": 1,
  "quality_self_check": {
    "completeness": 0-100,
    "gate_checks": ["gate1: PASS/FAIL", "gate2: PASS/FAIL"],
    "issues_found": ["问题"],
    "confidence": 0-100
  },
  "handoff_to": "next_stage_name",
  "handoff_ready": true/false,
  "timestamp": "ISO"
}
```

## Attempt Ledger 格式（主会话维护）

```json
{
  "task_id": "MSF-YYYYMMDD-NNN",
  "stages": {
    "scout": {"attempts": 1, "best_score": 85, "status": "PASS"},
    "analyze": {"attempts": 2, "best_score": 72, "status": "PASS"},
    "draft": {"attempts": 1, "best_score": 90, "status": "PASS"},
    "review": {"attempts": 1, "best_score": 82, "status": "PASS"},
    "revise": {"attempts": 1, "best_score": null, "status": "PASS"}
  },
  "total_attempts": 6,
  "final_score": 82,
  "delivery_status": "PASS"
}
```

## Authority Boundary

```yaml
ai_authority:
  - 文献采集与来源筛选
  - 证据分级与映射
  - PRISMA 流程生成
  - 初稿撰写与格式化
  - 审稿与质量评分
  - 修订与格式修正

human_authority:
  - 最终结论确认
  - 医学/战略决策
  - 对外发布
  - 临床建议
  - 伦理判断
  - ⏸️ STOP_FOR_HUMAN 标记的所有内容
```

## Recovery 规则

```
Stage N 返回 receipt.handoff_ready = false:
  → 主会话检查 issue 类型
  → 技术性失败（超时/工具错误）→ 直接重试（attempt+1）
  → 质量失败（gate 未过）→ 退回 Stage N-1，传入 issue list
  → 同一 stage 最多重试 3 次 → 超过则标记 HUMAN_ESCALATION
```

## 子代理文件

| Agent | 文件 | OPL 角色 |
|:-----:|:-----|:---------|
| 1 | agent1-scout.md | Stage 1: Scout |
| 2 | agent2-analyze.md | Stage 2: Analyze |
| 3 | agent3-draft.md | Stage 3: Draft |
| 4 | agent4-review.md | Stage 4: Independent Reviewer ⚠️ |
| 5 | agent5-revise.md | Stage 5: Revise |

## LEARNED PATTERNS

### v4.2.2: Review 评分标准化 + 二次Review闭环 (2026-05-22)
来源：两次MSF执行复盘（寰宇健康83分 vs 抗癌协会67.6分，标准不统一）
改动：
- agent4-review.md 新增评分矩阵（11项×权重，满分100，消除主观波动）
- agent5-revise.md 新增二次Review触发规则（original_score<80→re-review）
- SKILL.md 新增 Stage 5.5 Review 决策节点

### v4.2.1: web_fetch 反爬检测规则注入 (2026-05-22)
来源：MSF 尽职调研实战复盘
改动：agent1-scout.md 新增反爬关键词检测（验证码|captcha|滑块|403|404等），命中→自动切 babata-browser
触发：企查查/广东社会组织平台/BOSS直聘等反爬拦截导致 Scout 采集失败

### v4.2: OPL stage-led 注入 (2026-05-21)
来源：gaofeng21cn/one-person-lab + med-autoscience
吸收：Stage Receipts / Attempt Ledger / Per-Stage Gates / Review Ledger / Authority Boundary / Recovery
核心原则：AI-first quality gates require separation of labor — a single agent cannot execute and then review itself (MAS invariant)

### v4.1: 纸面造假修复 (2026-05-21)
v4.0 的 Agent 架构只存在于描述文字。补齐全 5 个 Agent prompt 文件。
