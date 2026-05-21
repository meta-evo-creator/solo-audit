---
name: deep-research
version: 3.2.0
description: |
  深度调研技能 v3.2 — File-based Handoff + Merge Agent + Stage State。Scout(3路并行)→Merge→Analyze(2路并行)→Draft→Review→Revise。设计来源：fainir/most-capable-agent-system-prompt(handoff.md)+GPT Researcher(planner/executor/publisher)+Magentic(task ledger)。
  Use when: 深度调研,技术评估,行业分析,多源交叉,L3任务
  NOT for: 简单搜索, 纪检巡视(use discipline-inspect)
metadata:
  openclaw:
    emoji: 🔍
    requires:
      sessions: sessions_spawn 可用
---

# Deep-Research v3.2 🔍

> 8-Agent File-based Handoff Pipeline。不再通过 task 参数传递数据——Agent 读写 `artifacts/` 目录。
> **v3.2: 吸收 fainir file-handoff + GPT Researcher Merge Agent + Magentic task ledger**

## 六阶段管线（8 Agent）

```
Phase 1: Scout (3路并行 sessions_spawn)
  ├─ Agent 1a: Web Scout → 写 artifacts/scout_web.json
  ├─ Agent 1b: Academic Scout → 写 artifacts/scout_academic.json
  └─ Agent 1c: Deep Scout → 写 artifacts/scout_deep.json

Phase 1.5: Merge (sessions_spawn) ← NEW
  └─ Agent 1d: Merge → 读 3 个 scout_*.json → 去重+评分→ 写 artifacts/merge.json + handoff.md

Phase 2: Analyze (2路并行 sessions_spawn)
  ├─ Agent 2a: 主视角 → 读 artifacts/merge.json → 写 artifacts/analyze_main.json
  └─ Agent 2b: 对立视角 → 读 artifacts/merge.json → 写 artifacts/analyze_counter.json

Phase 3: Draft (sessions_spawn)
  └─ Agent 3 → 读 artifacts/merge.json + analyze_*.json → 写 artifacts/draft_report.md

Phase 4: Review (sessions_spawn) ⚠️ 独立审计
  └─ Agent 4 → 读 artifacts/merge.json + draft_report.md → 写 artifacts/review_ledger.json

Phase 5: Revise (sessions_spawn)
  └─ Agent 5 → 读 artifacts/draft_report.md + review_ledger.json → 写 artifacts/final_report.md + revision_log.json
```

## File-based Handoff 协议

**核心原则（来自 fainir）：**
- "Write state to files, not just prompt context"
- "Resume from files rather than trusting long prompt history"
- "Leave an explicit handoff with next actions, blockers, open questions"

**artifacts/ 目录结构：**
```
artifacts/
  ├── handoff.md         ← 当前管线状态（谁完成、谁待执行、数据在哪）
  ├── status.md          ← 全局进度
  ├── scout_web.json     ← Agent 1a 输出
  ├── scout_academic.json← Agent 1b 输出
  ├── scout_deep.json    ← Agent 1c 输出
  ├── merge.json         ← Agent 1d 合并输出
  ├── analyze_main.json  ← Agent 2a 输出
  ├── analyze_counter.json← Agent 2b 输出
  ├── draft_report.md    ← Agent 3 输出
  ├── review_ledger.json ← Agent 4 输出
  ├── final_report.md    ← Agent 5 最终输出
  └── revision_log.json  ← Agent 5 修订日志
```

**handoff.md 格式：**
```markdown
# Pipeline Handoff
- Phase: Scout_COMPLETE / Merge_COMPLETE / ...
- Next: merge / analyze / draft / review / revise
- Input file: artifacts/merge.json
- Status: ready / blocked / failed
- Blockers: [如有]
- Last update: ISO timestamp
```

## 编排流程（主会话）

```
主会话创建 artifacts/ + 初始化 handoff.md
  │
  ├─ Phase 1: 并行 spawn 1a+1b+1c (context: isolated)
  │     各 Agent 写 artifacts/scout_*.json
  │
  ├─ Phase 1.5: spawn Agent 1d (merge)
  │     读 3 个 scout_*.json → 去重评分 → 写 artifacts/merge.json
  │
  ├─ Phase 2: 并行 spawn 2a+2b (context: isolated)
  │     读 artifacts/merge.json → 写 artifacts/analyze_*.json
  │
  ├─ Phase 3: spawn Agent 3 (draft)
  │     读 artifacts/merge.json + analyze_*.json → 写 artifacts/draft_report.md
  │
  ├─ Phase 4: spawn Agent 4 (review) ⚠️ 独立审计
  │     读 artifacts/merge.json + draft_report.md → 写 artifacts/review_ledger.json
  │
  ├─ Gate: score ≥ 80 → 交付
  │        score < 80 → Phase 5
  │
  └─ Phase 5: spawn Agent 5 (revise)
        读 draft_report.md + review_ledger.json → 写 artifacts/final_report.md
```

**主会话不传递数据——只做 spawn + gate 检查。**

## 子代理文件

| Agent | 文件 | v3.2 变化 |
|:-----:|:-----|:---------|
| 1a | agent1a-scout-web.md | 输出改为写 `artifacts/scout_web.json` |
| 1b | agent1b-scout-academic.md | 输出改为写 `artifacts/scout_academic.json` |
| 1c | agent1c-scout-deep.md | 输出改为写 `artifacts/scout_deep.json` |
| **1d** | **agent1d-merge.md** | **🆕 Merge Agent** |
| 2a | agent2a-analyze.md | 输入改为读 `artifacts/merge.json` |
| 2b | agent2b-analyze.md | 输入改为读 `artifacts/merge.json` |
| 3 | agent3-draft.md | 输入改为读 `artifacts/` 多文件 |
| 4 | agent4-review.md | 读 `artifacts/` → 写 `artifacts/review_ledger.json` |
| 5 | agent5-revise.md | 读/写 `artifacts/` |

## 与 v3.1 的关键差异

| 维度 | v3.1 | v3.2 |
|:-----|:-----|:-----|
| 数据传递 | task 参数手写摘要 | `artifacts/` 文件读写 |
| 合并 | 主会话手工 | **Agent 1d Merge** 自动去重评分 |
| 管线状态 | 无 | `handoff.md` + `status.md` |
| 信息损失 | 严重（三次摘要转述） | 零损失（文件原始 JSON） |
| 恢复能力 | 无（失败后从头来） | 读取 handoff.md + 续传 |
| Agent 数量 | 7 | **8** (+Merge) |

## LEARNED PATTERNS

### v3.2: File-based Handoff (2026-05-21)
来源: fainir/most-capable-agent-system-prompt + GPT Researcher + Magentic
v3.1 task 参数大小限制导致 30+ 来源摘要丢失 → v3.2 改为 artifacts/ 文件读写。
新增 Merge Agent 消除手工去重瓶颈。

### v3.1: 纸面造假修复 (2026-05-21)
v3.0 无子代理文件 → v3.1 补齐 7 个 Agent prompt。

### 审稿分离
Review Agent 独立会话，不继承上游上下文。
