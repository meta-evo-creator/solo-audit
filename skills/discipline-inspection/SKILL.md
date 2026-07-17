---
name: discipline-inspection
version: 1.3.0
description: |
  Discipline Inspection v1.3 ⚔️ 方法论v2.2·6核心模块(M2/M4/M6/M7/M8/M9)+基础两因素强制执行·聚焦纪检监察办案。⛔ Confidential.
platforms:
  - openclaw
tools:
  - ripgrep
  - sessions_spawn
  - memory_search
  - tavily_search
metadata:
  openclaw:
    emoji: ⚔️
---

# Discipline-Inspection ⚔️ 纪律审查 v1.0.1

> **纪律为准绳，持续守望。** 8-Agent File-based Handoff Pipeline + 违规+有责双因素分析法 + 二十四字方针6维Review评分矩阵。
> 与 DR v4.0 同构。
> ⛔ 保密，不上传 GitHub/ClawHub。仅限中山一院纪委办/监察处内部使用。

> 📎 共享配置：`skills/supervision-shared/shared-config.yaml`（WIKI路径/搜索链/模板）

---

## 🛡️ No-Authority Boundary（权限边界）

本技能是 **refs-only / no-authority** 能力包。

**本技能产出：**
- `violation_finding_ref` — 违规事实认定（候选）
- `evidence_chain_ref` — 证据链引用
- `article_match_ref` — 适用条款匹配
- `responsibility_assessment_ref` — 责任分析（候选）
- `sanction_recommendation_ref` — 处分建议（候选）
- `mitigation_aggravation_ref` — 从轻从重情节
- `owner_gate_handoff_ref` — 石冰确认门禁交接包

**本技能绝不产出：**
- 最终处分决定 · 问责结论 · 组织处理决定
- 对案件定性/量纪的最终判定
- 替代纪委会议/审批程序的任何产出

上述权限归 **石冰（领域所有者）** 及中山一院纪委办/监察处法定程序。

> 本边界声明对应 SOLO 655 铁律④（权责两清：最小代理——每项授权临时、有域、可撤销）。

---

## 📋 结构化输出引用（Ref Families）

本技能所有 Agent 产出通过结构化 ref 交接，替代自由格式报告。

**引用模板：** `references/big-oversight-ref-templates.md` ⚔️ DI 纪律审查 Ref 族

### 阶段→Ref 映射

| Phase | Agent | 产出文件 | Ref 族 |
|:------|:------|:---------|:-------|
| 0 | Scope | `agent0-scope.json` | `source_pack_ref` |
| 1a | Search-rg | `agent1a-search-rg.json` | `article_match_ref`（条款原文+出处） |
| 1b | Search-pkulaw | `agent1b-search-pkulaw.json` | `version_verified_ref` |
| 2 | Audit | `agent2-audit.json` | `evidence_chain_ref` + `article_match_ref`（版本审计后） |
| 3 | Analyze | `agent3-analyze.json` | `violation_finding_ref[]` + `responsibility_assessment_ref` + `mitigation_aggravation_ref` + `sanction_recommendation_ref` |
| 4 | Draft | `agent4-draft.md` | 上述 ref 综合 + 报告/提纲正文 |
| 5 | Review | `agent5-review_ledger.json` | 评分矩阵 + 修复建议（消费所有上游 ref） |
| 6 | Revise | `agent6-final.md` | 修正后的候选终版 |
| 7 | Publish | `agent6-final.md` | `owner_gate_handoff_ref`（汇总所有候选→石冰确认） |

### 交接规范

```
Agent 0 (source_pack_ref)
  → Agent 1a/1b (article_match_ref + version_verified_ref)
    → Agent 2 (evidence_chain_ref, 消费 1a+1b 的版本验证结果)
      → Agent 3 (violation_finding_ref + responsibility_assessment_ref + sanction_recommendation_ref)
        → Agent 4 (所有上游 ref 综合为报告/提纲)
          → Agent 5 (Review 评分矩阵消费所有 ref)
            → Agent 6 (修正)
              → Agent 7 (owner_gate_handoff_ref → 石冰)
```

> 每个 Agent 产出文件内应包含其 ref 的结构化字段（见模板详细定义）。主会话验证 ref 完整性（检查清单见模板），不读全文。

---

## ⛔ 入口阻塞（不可跳过·不可降级）

本技能**唯一入口**：Suit Phase 1 确认 → `sessions_spawn Agent 0(scope)`。

以下行为视为**违规执行** — 记录为 `[UNSOURCED-EXECUTION]`：
- 在主会话手动搜索法规
- 在主会话手动撰写分析/报告/底稿
- 凭记忆引用法规条款号
- 以「案件简单」「用户急用」为由跳过任何Agent

**本条款不因任何理由降级。**

---

## ⚡ Solo Status 协议（强制）

每次 Agent spawn 前 + 完成后，更新 `./solo/pipeline-status.json`。

```json
{
  "pipeline_id": "DI-YYYYMMDD-xxx",
  "skill": "discipline-inspection",
  "topic": "问题摘要",
  "started_at": "ISO时间",
  "last_updated": "ISO时间",
  "mode": "full / interview / quick",
  "phases": {
    "0: Scope":     {"status": "completed", "detail": "问题界定"},
    "1a: Search-rg":  {"status": "running",   "detail": "rg WIKI搜索中"},
    "1b: Search-pkulaw": {"status": "pending",   "detail": "pkulaw版本验证"},
    "2: Audit":     {"status": "pending",   "detail": ""},
    "3: Analyze": {"status": "pending",   "detail": ""},
    "4: Draft":   {"status": "pending",   "detail": ""},
    "5: Review":  {"status": "pending",   "detail": ""},
    "6: Revise":  {"status": "pending",   "detail": ""},
    "7: Publish": {"status": "pending",   "detail": ""}
  }
}
```

---

## 八阶段管线（按模式分流）

```
Phase 0: Scope     → Agent 0: 问题界定             → agent0-scope.json
Phase 1a: Search-rg → Agent 1a: rg WIKI法规+案例搜索 → agent1a-search-rg.json
Phase 1b: Search-pkulaw → Agent 1b: pkulaw版本验证   → agent1b-search-pkulaw.json
Phase 2: Audit     → Agent 2: 法规引用审计(合并1a+1b) → agent2-audit.json
Phase 3: Analyze → Agent 3: 深度分析               → agent3-analyze.json
Phase 4: Draft   → Agent 4: 撰写报告/提纲          → agent4-draft.md
Phase 5: Review  → Agent 5: 内容质量审计           → agent5-review_ledger.json
Phase 6: Revise  → Agent 6: 修复                   → agent6-final.md + revision_log.json
Phase 7: Publish → 主会话: solo-file-transfer      → IMA知识库
```

**每个 Agent 独立隔离会话 (context:isolated, lightContext:true)。主会话只做 spawn + gate + 文件验证。**

> 🔒 **降熵契约生效（`skills/solo/SKILL.md §2.3`）：** 子代理回传压缩为 DONE+路径+≤3行摘要。文件验证只检查存在性，不读全文。Draft/分析阶段强制子会话隔离执行。
> lightContext 跳过全量 bootstrap 注入以压缩 token，仅加载该 Agent 所需的单文件 prompt。

---

## Guardrail Routing：任务分流

| 模式 | 触发场景 | 管线 | Agent数 |
|:-----|:---------|:-----|:--:|
| **full** | 案件定性、处分建议 | 0→1a→1b→2→3→4→5→6→7 | 8+1 |
| **interview** | 谈话提纲 | 0→1a→1b→2→3+4→7 (Analyze+Draft合并) | 5+1 |
| **quick** | 法规咨询、条款查询 | 0→1a→1b→2→7 | 4+1 |

**分流决策点：** Agent 0 Scope 完成后，主会话根据 `task_type` 选择模式。

**设计原则：** Anthropic 6大模式之 Guardrail-Routed Architecture — 根据任务类型而非复杂度决定处理深度。

---

## 🔒 文件落地验证（强制）+ 降熵约束

每个 Agent spawn 完成（状态=done）后，主会话执行文件存在性检查。

**验证规则：**
```
read <output_file_path> → 检查文件存在且size > 0
# ⛔ 只验证存在性，不读文件内容到主会话上下文
```

**文件不存在 → 标记该 Agent 为 `failed` → 不进入下一阶段 → 向石冰报告具体失败原因。**
**文件存在 → 仅记录路径 + 3行摘要 → 进入下一阶段。**

---

## 输出路径协议

```
C:\Users\shibi\.openclaw\workspace\memory\inspection-drafts\{task_id}\
├── agent0-scope.json          ← Phase 0: 问题界定
├── agent1a-search-rg.json     ← Phase 1a: rg WIKI法规+案例+方法论 (含source_line)
├── agent1b-search-pkulaw.json ← Phase 1b: pkulaw版本验证 (含version_verified)
├── agent2-audit.json          ← Phase 2: 法规引用审计 (合并1a+1b)
├── agent3-analyze.json        ← Phase 3: 分析推理 (interview模式=分析+提纲草稿)
├── agent4-draft.md            ← Phase 4: 正式报告/底稿 (full模式)
├── agent5-review_ledger.json  ← Phase 5: 内容质量审计 + PASS/WARN/FAIL
├── agent6-final.md            ← Phase 6: 终版 (full模式)
└── revision_log.json          ← Phase 6: 修订日志
```

**task_id格式：** `DI-YYYYMMDD-序列`

**主会话不传递数据 — 只做 spawn + gate + 文件验证。** 同DR的file-based handoff协议。

**降熵约束（强制）：** 文件验证只检查存在性。各Agent的详细产出不进入主会话上下文。Draft阶段必须 spawn 子代理隔离执行。

---

## 各 Agent 详细规格

---

### Agent 0: Scope（问题界定）

**输入：** 用户提供的案件事实
**输出：** `agent0-scope.json`

**产出物结构：**
- `case_summary` — 对象、行为、金额、时间跨度
- `legal_framework` — 适用法规清单（无需条款号）+ 关键法律问题
- `evidence_assessment` — 已有证据 + 缺失证据 + 策略方向
- `interview_strategy_framework` — 若为谈话提纲则提供策略方向
- `risk_assessment` — 主要争议点和风险
- `task_type` — 判定任务类型（case_qualification / interview_outline / legal_consultation）
- `downstream_handoff` — 给Agent 1的搜索关键词建议 + 给Agent 3的分析方向

```json
{
  "task_id": "DI-YYYYMMDD-xxx",
  "task_type": "case_qualification | interview_outline | legal_consultation",
  "case_summary": {
    "subject": "涉案人员身份",
    "behavior": "涉案行为描述",
    "amount": "涉案金额",
    "time_span": "时间跨度"
  },
  "legal_framework": {
    "applicable_laws": ["法规名称列表"],
    "key_legal_questions": ["关键法律问题"]
  },
  "evidence_assessment": {
    "available": ["已有证据"],
    "missing": ["缺失证据"],
    "strategy": "取证策略方向"
  },
  "interview_strategy_framework": "若为谈话提纲则提供策略方向",
  "risk_assessment": "主要争议点和风险",
  "downstream_handoff": {
    "agent1_search_terms": ["搜索关键词"],
    "agent3_analysis_direction": "分析方向"
  }
}
```

## ⚠️ 产出规则
写文件输出，最终回复仅一行 `DONE <输出文件路径>`。
不要复述文件内容、不要写摘要。主会话会自己读文件。

---

### Agent 1: Search（法规搜索）⚠️ 行为强制约束

**输入：** `agent0-scope.json`
**输出：** `agent1a-search-rg.json + agent1b-search-pkulaw.json`

**⛔ 搜索行为强制约束：**

```
Step 1 [MANDATORY·不可跳过]:
  ripgrep 纪律法规库 + 医药行为规范 + 指导性案例 → 法规/案例原文
  rg -n "关键词" C:\Users\shibi\.openclaw\wiki\main\sources\discipline\法规\
  rg -n "关键词" C:\Users\shibi\.openclaw\wiki\main\sources\medical\
  rg -n "关键词" C:\Users\shibi\.openclaw\wiki\main\sources\discipline\指导性案例\
  rg -n "关键词" C:\Users\shibi\.openclaw\wiki\main\sources\hospital-inspection\
  rg -n "关键词" C:\Users\shibi\.openclaw\wiki\main\sources\inspection\
  必须产出: 法规/案例原文(一字不差) + 来源文件路径

  核心法规（已确认版本）:
    中国共产党纪律处分条例_2023修订
    中华人民共和国监察法_2024修正
    监察法实施条例_2025修订
    监督执纪工作规则_2019
    事业单位工作人员处分规定_2023
    中华人民共和国公职人员政务处分法2020.6.20
    中华人民共和国刑法（根据修正案十一修正，2020年）

  医药行业规范（医德医风方向优先引用）:
    医疗机构从业人员行为规范（2012，10章60条完整原文）
    医疗机构工作人员廉洁从业九项准则（2021，国卫医发37号）
    医务人员职业道德准则2025年版（2025，国卫医政发9号，四部门联合发布）

  时间预算: 1-2分钟

Step 1A [省级法规搜索·P-002复用于DI·不可跳过]:
  ⚠️ 搜索范围必须覆盖 **国家→省级→院内** 三级法规链
  ⚠️ 任务涉及广东省机构和人员时，必须搜索省级法规:
    rg -n "关键词" C:\Users\shibi\.openclaw\wiki\main\sources\inspection\ --include "*粤府令*" --include "*广东省*" --include "*地方性法规*" -i
    ⚠️ 若rg未命中 → web_search补充: search "site:gov.cn 广东省 [法规领域] 办法"
  获取全文后下载至 wiki/main/sources/inspection/ 再引用
  时间预算: 1-2分钟

Step 1B [版本验证·2026-07-15新增·不可跳过]:
  ⚠️ WIKI中命中的法规，**必须通过 pkulaw-search 确认版本为现行有效**后才能引用
  ① 对每部WIKI命中的核心法规:
    python skills/pkulaw-search/scripts/pkulaw_search.py law --title "法规名" --json
    → 检查 timeliness/发布日期/施行日期
    → version字段与WIKI frontmatter对照
  ② 版本不一致 → 标记 [VERSION_OUTDATED] → 触发 regulation-manager update
  ③ 输出到 agent1a-search-rg.json + agent1b-search-pkulaw.json 的 version_verified 字段
  时间预算: 1-2分钟

Step 2 [仅当Step 1+1A+1B覆盖不足]:
  执行（按优先级）:
  ① Tavily/web_search → 政府网站法规搜索（绕过WAF获取索引缓存全文）
     search "site:gov.cn OR site:ccdi.gov.cn 法规名 文号"
  ② ①仍不满足 → babata-search 最新指导性案例/方法论文
     cd C:\Users\shibi\.openclaw\workspace\skills\babata-search\scripts; node search.js baidu "关键词"
  限制: 最多3组关键词
  时间预算: 2-3分钟

Step 3 [WIKI缺失/版本滞后·触发RM入库]:
  ⚠️ Step 1未命中或Step 1B发现版本滞后 → 触发 regulation-manager 技能:
    RM add → PKULaw MCP确认 → 全文下载 → 规范化 → WIKI入库
    RM update → 新旧对比 → 替换旧版

⛔ 禁止:
  - 跳过Step 1直接做web搜索
  - 跳过Step 1B直接引用WIKI中的法规（未确认版本时效性）❗2026-07-15
  - 用web_fetch打开百度/微信文章
  - 凭记忆引用条款号/案例
  - 条款号须从rg输出中直接提取，不得自行编造/推算条款编号（❗幻觉防范——若rg输出无条款号则标注'[条款号待确认]'）

📡 `[UNCERTAIN]` 标记协议（数据溯源链·P-001复用于DI）:
  - 从web_fetch/web_search获取的非官方源数据（来自第三方网站而非gov.cn/ccdi.gov.cn）→ 标注`[UNCERTAIN: 来源非官方]`
  - quantifiable数据（金额、数量、比例等）无法溯源官方源或来自推测估算 → 标注`[UNCERTAIN: 推测数据]`
  - 标注`[UNCERTAIN]`的数据项，下行Agent 2 (Audit) 必须将其列为`unsourced_claims`，**禁止**再作为定量计算参数传递给Agent 3 (Analyze)

📡 搜索来源优先级（P-003复用于DI）:
  法规来源: gov.cn > ccdi.gov.cn > flk.npc.gov.cn > 官方公报 > 第三方转载
  案例来源: 中央纪委国家监委官网 > 省级纪委监委官网 > 裁判文书网 > 第三方整理

```

**产出物结构：**
- `legal_provisions` — 每条含 `law/article/text_exact/source_file/applicability`
- `guiding_cases` — 指导性案例（批次+编号+核心事实+处理结论+参考价值）
- `methodology_notes` — 方法论文要点（违规+有责两因素分析等）
- `penalty_benchmarks` — 处分档次对照
- `search_log` — 搜索路径→结果追踪

```json
{
  "legal_provisions": [
    {
      "law": "法规名",
      "article": "条款号",
      "text_exact": "原文",
      "source_file": "绝对路径",
      "applicability": "适用性"
    }
  ],
  "guiding_cases": [
    {
      "batch": "批次",
      "case_id": "编号",
      "core_facts": "核心事实",
      "conclusion": "处理结论",
      "reference_value": "参考价值"
    }
  ],
  "methodology_notes": [
    "方法论要点"
  ],
  "penalty_benchmarks": {
    "处分档次对照": {}
  },
  "total_clauses": 0,
  "total_cases": 0,
  "search_log": [
    "搜索追踪记录"
  ]
}
```

## ⚠️ 产出规则
写文件输出，最终回复仅一行 `DONE <输出文件路径>`。

---

### Agent 2: Audit（法规引用审计）

**输入：** `agent0-scope.json` + `agent1a-search-rg.json + agent1b-search-pkulaw.json`
**输出：** `agent2-audit.json`

**审计清单：**
1. **条款号原文验证** — 每条法规引用回原文比对（用ripgrep二次确认）
2. **`[UNCERTAIN]` 阻断检查**（P-001复用于DI）:
   - 扫描 agent1a-search-rg.json + agent1b-search-pkulaw.json 全文中的 `[UNCERTAIN]` 标记
   - 含有 `[UNCERTAIN]` 标记的数据项 → 移入 `unsourced_claims` 数组
   - 若 `guiding_cases` 或关键数据中任一项含 `[UNCERTAIN]` → 在结论中标记 `BLOCK_DOWNSTREAM: <数据项>`
   - **Agent 3 (Analyze) 禁止将 `unsourced_claims` 中的数据项用作定量计算参数**
3. **主体-行为-结果三要素验证** — 每条引用条款逐条检查：
   ✅ 主体要件：涉案人是否属于该条款的适用对象范围？
   ✅ 行为要件：条款描述的行为是否匹配涉案行为？
   ✅ 结果要件：处分/处罚是否在本案可行范围内？
   三要素不全的条款降级标注为「参照」。
4. **版本一致性** — 所有法律引用是否使用了最新版本
5. **金额门槛准确性** — 刑事立案标准、党纪处分对应金额是否正确
6. **案例来源完整性** — 指导性案例是否标注批次、编号、发布机关

**结论：PASS / PASS_WITH_WARNINGS / FAIL**

```json
{
  "audit_conclusion": "PASS | PASS_WITH_WARNINGS | FAIL",
  "checks": [
    {
      "type": "条款验证",
      "regulation": "法规+条款",
      "matched": true,
      "verified_text": "原文",
      "source": "路径"
    }
  ],
  "version_issues": [
    {
      "regulation": "法规名",
      "used_version": "使用版本",
      "current_version": "当前最新版本"
    }
  ],
  "unsourced_claims": [],
  "block_report": {
    "blocked_items": ["被阻断的数据项列表"],
    "downstream_blocked": true
  },
  "issues": [
    {
      "severity": "critical | high | medium | low",
      "description": "问题描述",
      "fix": "修复建议"
    }
  ]
}
```

**fail时 → 阻塞管线，问题返回Agent 1修正后重新Audit。**

## ⚠️ 产出规则
写文件输出，最终回复仅一行 `DONE <输出文件路径>`。

---

### Agent 3: Analyze（深度分析·违规+有责双因素分析法）

**输入：** `agent0-scope.json` + `agent1a-search-rg.json + agent1b-search-pkulaw.json` + `agent2-audit.json`
**输出：** `agent3-analyze.json`（full模式）/ `agent4-draft.md`（interview模式直接写提纲）

**分析方法论：违规+有责双因素分析框架**

> 方法论来源: `wiki/main/sources/discipline/方法论/违规+有责两因素分析方法论.md`
> 嵌入自中纪委执纪执法指导性案例方法论。

```
┌───────────────────────────────────────────────────────────┐
│            违规+有责 双因素分析框架                         │
│                                                           │
│ 因素一：违规（客观行为要素）                                │
│ ├─ 行为事实：涉案人实施了什么行为？                         │
│ ├─ 法律依据：行为违反了哪些党纪/法规条款？                   │
│ ├─ 客体侵害：行为侵害了什么纪律/法律保护的利益？              │
│ ├─ 行为持续性：一次性/偶发 vs 系统性/持续性？                │
│ └─ 情节程度：涉案金额·频次·影响范围·社会后果                │
│                                                           │
│ 因素二：有责（主观归责要素）                                │
│ ├─ 主观状态：故意/过失？直接/间接？                         │
│ ├─ 认知程度：是否明知违规？是否可合理期待知晓？               │
│ ├─ 动机目的：个人牟利/组织利益/外部压力？                    │
│ ├─ 是否主动纠错：主动停止·主动报告·退赃挽回？                │
│ └─ 身份认知：是否清楚自身党员干部身份对应的义务？             │
│                                                           │
│ 综合判定 = 违规（定性·程度） × 有责（轻重）                  │
│ ├─ 违规成立 + 有责成立 → 违纪/违法成立                      │
│ ├─ 违规成立 + 有责不成立 → 不构成违纪                       │
│ └─ 违规∧有责 → 量纪档次 = f(违规程度, 有责程度)             │
└───────────────────────────────────────────────────────────┘
```

**full模式分析维度：**
- 违规因素分析（客观行为→对应条款→情节评价）
- 有责因素分析（主观状态→动机目的→事后态度→身份叠加）
- 综合量纪建议（违规程度×有责程度→处分档次区间）
- 对比案例参照（指导性案例中的违规/有责认定路径）
- 证据链完整性评估（哪些违规/有责要素有证据支撑、哪些依赖口供）

**每条引用法规必须附 `[适用性论证]` 字段。**

**🔴 事实前提声明（premise_declaration）— 大监督强制（Step 0b）:**

Agent 3 必须在分析产出中显式声明分析依赖的 **3个最强前提假设**（最容易想当然的），并逐一执行 rg 工具验证。这是 Step 0b 事实门禁在 Agent 3 的结构化落地。

**填写规则：**
1. **前提1（强制）：涉案主体的法规身份认定** — 明确被审查人是「公职人员」「中共党员」「事业单位工作人员」「普通职工」还是其他身份类别，附 rg 验证来源（检索 wiki 法规原文中对主体要件的规定）
2. **前提2-3（强制）：分析推理中最关键的 2 个事实假设** — 各附 rg/web_search 验证来源
3. **sensitivity 字段**：说明若任一前提不成立，结论将如何变化

**🔴 对抗论证（counter_argument）— 防止确认偏误:**

Agent 3 必须在分析中构造最强对抗观点并逐条驳回。这是防止单向论证偏误的结构化保障。

**填写规则：**
1. **strongest_opposing_view**：最有利于被审查人的免责/减责理由（不要弱化，写成最强形态）
2. **why_this_view_is_rejected**：该理由被驳回的具体原因（引用法规+事实）
3. **residual_uncertainty**：即使驳回后仍存在的剩余不确定性

```json
{
  "analysis": {
    "violation_factor": {
      "behavior_description": "涉案行为客观描述",
      "applicable_regulations": [
        {
          "law": "法规名",
          "article": "条款号",
          "violation_type": "违规类型",
          "applicability_argument": "[适用性论证]"
        }
      ],
      "protected_interests": "受侵害的纪律/法律保护利益",
      "continuity": "一次性/偶发/持续性/系统性",
      "severity": {
        "amount": "涉案金额",
        "frequency": "频次",
        "scope": "影响范围",
        "consequence": "社会后果"
      }
    },
    "responsibility_factor": {
      "subjective_state": "直接故意/间接故意/疏忽过失",
      "knowledge_level": "明知/应知/不可合理期待知晓",
      "motive": "个人牟利/单位利益/外部压力/其他",
      "post_behavior": "主动纠错退赃/消极被动/对抗不配合",
      "identity_weight": "党员干部身份对注意义务的叠加影响"
    },
    "comprehensive_assessment": {
      "violation_established": true,
      "responsibility_established": true,
      "penalty_range": "处分档次区间",
      "aggravating_factors": ["从重因素"],
      "mitigating_factors": ["从轻因素"],
      "recommended_disposition": "建议处分档次"
    }
  },
  "premise_declaration": {
    "assumptions": [
      "前提1（身份认定）：涉案主体法规身份为______ 【来源：rg验证：______】",
      "前提2：______ 【来源：rg验证：______】",
      "前提3：______ 【来源：rg验证：______】"
    ],
    "sensitivity": "若前提不成立，结论将变化"
  },
  "counter_argument": {
    "strongest_opposing_view": "最有利免责理由",
    "why_this_view_is_rejected": "不成立原因",
    "residual_uncertainty": "剩余不确定性"
  },
  "case_references": [
    {
      "case_id": "案例编号",
      "similarity": "相似度",
      "reference_value": "参考价值"
    }
  ],
  "evidence_chain": [
    {
      "element": "违规/有责要素",
      "evidence": ["支撑证据"],
      "gap": "依赖口供/已有证据"
    }
  ],
  "unsourced_claims": 0,
  "confidence": "high | medium | low"
}
```

## ⚠️ 产出规则
写文件输出，最终回复仅一行 `DONE <输出文件路径>`。

⏸️ **STOP_FOR_REVIEW**
向石冰展示分析阶段产出摘要（审计发现TOP 5 + 风险排序 + 初步结论），等待回复。
- 石冰回复「继续」或超时15分钟 → 用当前优先级进入Draft
- 石冰调整优先级 → 按反馈重排序后进入Draft

---

### Agent 4: Draft（撰写报告/底稿）— full模式

**输入：** `agent0-scope.json` + `agent3-analyze.json`（+ agent2-audit.json）
**输出：** `agent4-draft.md`

**报告结构（按task_type调整）：**
- 一、案件定性框架（行为性质+法律适用表）
- 二、证据分析（盘点+策略）
- 三、量纪建议 / 谈话策略+问话脚本+预案
- 四、处理建议（场景分级）
- 🔒 G4.5 执行签名

## ⚠️ 产出规则
写文件输出，最终回复仅一行 `DONE <输出文件路径>`。

---

### Agent 5: Review（内容质量审计·二十四字方针6维评分矩阵）— full模式

**输入：** `agent4-draft.md` + `agent1a-search-rg.json + agent1b-search-pkulaw.json`
**输出：** `agent5-review_ledger.json`

**设计来源：** 纪检监察案件审理「二十四字方针」— 事实清楚·证据确凿·定性准确·处理恰当·手续完备·程序合规。

**评分矩阵（6维×权重）：**

| # | 维度 | 权重 | 角色 | 检查内容 |
|:-:|:-----|:----:|:-----|:---------|
| 1 | **定性准确** | **25%** | 核心 | 法规引用完整？原文一字不差？三要素匹配？违规+有责双因素完整？ |
| 2 | **事实清楚** | **20%** | 前提 | 行为链条完整？时空/金额/次数/手段/目的关联清晰？ |
| 3 | **证据确凿** | **20%** | 支撑 | 已有+缺失证据盘点完整？间接证据链路径可行？ |
| 4 | **处理恰当** | **15%** | 输出 | 量纪建议匹配法规和事实？情景矩阵充分？对抗论证到位？ |
| 5 | **手续完备** | **10%** | 保障 | 谈话程序规范？权利义务告知？各阶段完整？ |
| 6 | **程序合规** | **10%** | 底线 | 谈话策略符合法定程序？突破口不越权？取证路径合法？ |
| **合计** | | **100%** | | |

**评分标准：**
- 总分 ≥ 80 → PASS（跳过Phase 6）
- 总分 60-79 → REVISE（进入Phase 6）
- 总分 < 60 → REJECT（退回Agent 4重写，最多2轮，超限→HUMAN_ESCALATION）

**降档规则：**
- 定性准确 < 50 → 强制 REVISE
- 事实清楚 < 40 → 强制 REVISE
- 证据确凿 < 40 → 强制 REVISE

```json
{
  "review_score": 0,
  "verdict": "PASS | REVISE | REJECT",
  "score_breakdown": {
    "1_定性准确": {
      "score": 0,
      "weight": 25,
      "weighted": 0
    },
    "2_事实清楚": {
      "score": 0,
      "weight": 20,
      "weighted": 0
    },
    "3_证据确凿": {
      "score": 0,
      "weight": 20,
      "weighted": 0
    },
    "4_处理恰当": {
      "score": 0,
      "weight": 15,
      "weighted": 0
    },
    "5_手续完备": {
      "score": 0,
      "weight": 10,
      "weighted": 0
    },
    "6_程序合规": {
      "score": 0,
      "weight": 10,
      "weighted": 0
    }
  },
  "downgrade_triggers": [],
  "issues": [
    {
      "severity": "critical | high | medium | low",
      "dimension": "1-6",
      "description": "问题描述",
      "fix": "修复建议"
    }
  ],
  "must_fix": [],
  "sourced_claims": 0,
  "unsourced_claims": 0
}
```

## ⚠️ 产出规则
写文件输出，最终回复仅一行 `DONE <输出文件路径>`。

---

### Agent 6: Revise（修复）— full模式

**输入：** `agent4-draft.md` + `agent5-review_ledger.json`
**输出：** `agent6-final.md` + `revision_log.json`

基于Review的 `must_fix` 逐项修复。每项修复记录到 `revision_log.json`。

**修复完成后 → Review 可选二次验证（若原始score < 70）。**

## ⚠️ 产出规则
写文件输出，最终回复仅一行 `DONE <输出文件路径>`。

---

### Phase 7: Publish（IMA上传）

**主会话直调，不 spawn 子代理：**

```bash
node skills/solo-file-transfer/scripts/ima-upload.cjs <终版文件> <KB_ID>
```

- full模式: agent6-final.md
- interview模式: agent4-draft.md
- quick模式: agent2-audit.json（经审计验证的搜索结果）

**⛔ 报告纯净原则：**
- 上传 IMA 的报告必须是**纯分析内容**，禁止包含管线ID/Agent标识/审计声明JSON/VERIFIED清单等元数据
- 元数据写入 `memory/inspection-drafts/<case>/` 目录下的独立trace文件
- 报告只保留：标题、日期、执行摘要、分析正文、法规引用（附条款号+原文）

**常用KB_ID：** 见 MEMORY.md → IMA知识库段。

---

## 脱敏协议

子Agent prompt中涉及的组织名称使用脱敏替代符：
- 医院: 「某三甲医院」「某省属医院」
- 具体敏感数据不直接写入prompt，引用Agent 0 scope文件路径

---

## 法规知识库（可插拔Provider架构 🔌 v1.4.0）

> 法规数据层通过 Provider 接口与管线解耦。详见 `providers/regulation-source.interface.md`。

### Provider 自动检测

管线启动时按以下优先级选择知识源：
1. `WIKI_PATH` 环境变量存在 → wiki-provider（完整45+部法规）
2. `pkulaw-mcp` 可用 → 叠加 pkulaw-provider（版本验证）
3. 以上均不可用 → default-provider（3部核心法规demo）

### 可用 Provider

| Provider | 法规检索 | 版本验证 | 案例搜索 | 适用场景 |
|:---------|:-------:|:-------:|:-------:|:---------|
| **default-provider** | 3部核心 | ❌ | ❌ | 开源用户开箱即用 |
| **wiki-provider** | 45+部全文 | ❌ | 11个案例 | 有WIKI库的机构 |
| **pkulaw-provider** | ✅ | ✅ | ❌ | 有北大法宝订阅 |

### 降级行为

- 无 wiki-provider → default-provider 兜底，产出标记 `⚠️ 仅3部核心法规`
- 无 pkulaw-provider → Agent 1b 降级产出 `VERSION_UNVERIFIED`，管线不阻断
- 完全无知识源 → 管线拒绝启动

### 法规清单

**纪律法规**（`${WIKI_PATH}/discipline/法规/`，共45部）:
- 中国共产党纪律处分条例_2023修订
- 中华人民共和国监察法_2024修正
- 监察法实施条例_2025修订
- 监督执纪工作规则_2019
- 事业单位工作人员处分规定_2023
- 中华人民共和国公职人员政务处分法2020.6.20
- 中华人民共和国刑法（根据修正案十一修正，2020年）
- 中国共产党问责条例_2019修订
- 其他法规共计45部

**医药行为规范**（`${WIKI_PATH}/medical/`，共8部）:
- 医疗机构从业人员行为规范
- 医疗机构工作人员廉洁从业九项准则
- 医务人员职业道德准则2025年版
- 中华人民共和国医师法
- 中华人民共和国基本医疗卫生与健康促进法
- 中华人民共和国药品管理法
- 医务人员互联网健康科普负面行为清单
- 医药代表管理办法-2026

**指导性案例**（`${WIKI_PATH}/discipline/指导性案例/`，共11个）:
- 案例-公车私用私车公养
- 案例-容错纠错-两因素分析
- 案例-对抗组织审查-两因素分析
- 案例-形式主义官僚主义
- 案例-空白公函贪污
- 案例-违规吃喝公私混合
- 案例-违规摊派
- 案例-违规操办婚丧喜庆
- 案例-退休后违规接受宴请
- 案例-问责简单泛化
- 案例-骗领惠民惠农补贴

**分析方法论：**
- `${WIKI_PATH}/discipline/方法论/违规+有责两因素分析方法论.md`

---

## LEARNED PATTERNS

### v1.4.0 — 三层解耦：知识源可插拔Provider架构 (2026-07-17)
**来源：** 石冰将DI上传GitHub后发现外部用户无法使用——缺少WIKI法规库和pkulaw-mcp。
**核心设计：**
- 法规数据层通过 Provider 接口与管线解耦
- 新建 `providers/` 目录：接口规范 + default/wiki/pkulaw 三个 Provider
- Agent 1a/1b 去硬编码路径，改用环境变量 `${WIKI_PATH}` / `${SKILL_DIR}`
- Agent 1b 新增降级路径：pkulaw不可用时全标 `VERSION_UNVERIFIED`（不阻断管线）
- Agent 2 新增 `degradation_mode` 判定：降级模式 PASS→PASS_WITH_WARNINGS
- 默认知识包：3部核心法规全文（纪律处分条例+监察法+政务处分法）+ 方法论文档
- `shared-config.yaml` 环境变量化，去 `C:\Users\shibi` 硬编码
**原则：** 管线方法论是产品，法规数据是燃料——分开交付，各自配置。
**文件：**
- 新增: `providers/` (接口 + 3个provider配置 + 默认知识包)
- 新增: `README.md` (开源项目首页)
- 修改: `agents/search-rg.md`, `agents/search-pkulaw.md`, `agents/audit.md`
- 修改: `SKILL.md` (Provider架构文档), `supervision-shared/shared-config.yaml`

### v1.1.0 — Agent 1a/1b 拆分：pkulaw 结构性不可跳过 (2026-07-16)
**来源：** DI-20260716-001 复盘 — Agent 1 跳过 Step 1B 根因分析。即使加固了指令约束，单 Agent 内 rg+pkulaw 仍可能被跳过。拆分为 1a(rg WIKI) → 1b(pkulaw) 两个独立子会话后，1b 结构性独立存在——不可能被 1a 跳过。
**修改：**
- 新增 `agents/search-rg.md` (Agent 1a)：专注 rg WIKI搜索 + source_line + regulation_list
- 重构 `agents/search.md` → 重命名用意 `agents/search-pkulaw.md` 未执行，改为新建 (Agent 1b)：专注 pkulaw version_verified
- 更新 `agents/audit.md`：Gate 1 检查 1b 文件存在性 + Gate 2 检查 1a source_line
- SKILL.md 管线从 0→1→2→... 改为 0→1a→1b→2→...
- 全模式 Agent 数 +1（full: 7→8, interview: 4→5, quick: 3→4）
**原则：** 高风险的子步骤拆分为独立子会话 = 结构性不可跳过。效率不变（pkulaw 是瓶颈，rg 串行 30s 不增加总耗时）。

### v1.0.4 — Agent 1/2 agent文件与SKILL.md内容断层修复 (2026-07-16)
**来源：** DI-20260716-001 复盘。Agent 1 跳过 Step 1B（pkulaw-search版本验证），且编造了纪律处分条例不存在的第36-37条（虚假法条幻觉）。根因：agents/search.md 和 agents/audit.md 是 SKILL.md 的精简版，缺失了 Step 1B 强制指令、条款号防幻觉规则、前置门禁等关键执行约束。
**修复：**
- agents/search.md: 补充 Step 1B pkulaw-search 强制执行段（含 command + version_verified 必填字段）、Step 1A 省级法规、条款号防幻觉规则（source_line 必填）、[UNCERTAIN] 标记协议
- agents/audit.md: 增加前置门禁（Gate 1: Step 1B 存在性检查→直接FAIL / Gate 2: source_line 完整性→UNSOURCED计数→超阈值FAIL）、HALLUCINATION检测规则、[UNCERTAIN]阻断检查
**原则：** agent 文件不能是 SKILL.md 的"精简版"——SKILL.md 中的强制执行约束必须在 agent 文件中逐条落地为结构化检查。精简=削除护栏。

### v1.0.1 — 从DI v3.0.2独立剥离 (2026-06-08)
**来源：** 原 discipline-inspect v3.0.2 分拆。纪律审查与巡视监督方法论不兼容（违规+有责 vs 政治体检），Wiki数据层已独立（discipline/ vs inspection/），DI-20260603-xunshi 已做独立验证。
**核心继承：**
- 8阶段管线 + DR同构架构 ✓
- Guardrail Routing (full/interview/quick) ✓
- 违规+有责双因素分析框架 ✓
- 二十四字方针6维评分矩阵 ✓
- 文件落地验证 + 输出路径协议 ✓
- 脱敏协议 ✓

### v1.0.1 — quick模式必须包含Audit (2026-07-06)
**来源：** 石冰纠正：quick模式（法规咨询/条款查询）不可跳过Agent 2 Audit。
所有引用的法规条款必须经ripgrep原文比对验证后才可进入Publish阶段。
**修改：** quick模式管线从 `0→1→7` 修正为 `0→1→2→7`。
**移除项：**
- 巡视工作条例从核心法规列表移除（已属于supervision-inspection）
- 巡视制度wiki路径从Agent 1搜索范围移除
- xunshi模式从Guardrail路由移除

### v1.0.2 — 处分审批分类分析原则 (2026-07-06)
**来源：** 石冰纠正 — DI-20260706-001中「所有党纪处分必须上党委会」表述不精确。警告/严重警告可由同级纪委审查批准（《批准权限和程序规定》第6条），不必上党委会。
**教训：** 涉及处分审批程序的问题，必须按**轻处分（警告/严重警告）vs 重处分（撤销党内职务以上）**两个维度拆解检索和分析。
**交叉验证要求：** 涉及审批程序的问题必须**同时**检索以下两部法规并做交叉比对：
  - 《处分违纪党员批准权限和程序规定》（2022）→ 审批权限划分
  - 《监督执纪工作规则》（2019）→ 集体讨论形式
**输出规范：** '集体讨论决定'≠'上党委会'——必须在结论中区分「纪委常委会集体讨论」与「党委会集体讨论」的适用场景。
**绝对表述检查：** 凡使用「必须/应当/所有/一律」等绝对词，须先自问：是否存在例外情形？如有，标注例外条件。

### 从DI继承的教训
**主体-行为-结果三要素验证（2026-05-28）：**
每条引用条款须逐条检查主体要件、行为要件、结果要件，三要素不全的降级标注为「参照」。

**文件落地验证（2026-05-27）：**
每个Agent完成后主会话read验证文件存在性，不存在则标记failed。

**Suit硬执行：**
纪律审查/案件定性/谈话提纲等关键词 → 第一步展示确认提示 → 石冰确认 → sessions_spawn Agent 0。

> ⛔ 本技能为保密专属，仅本地使用，不得上传 GitHub/ClawHub 或任何外部平台。

### v1.0.3 �� Agent 0 Step 0b ǰ��������֤ǿ�� (2026-07-08)
**��Դ��** DI-20260708-001 ��Ʒ��� �� Agent 0 δִ��rg��֤���ٴ�ҽ���궨Ϊ'��ְ��Ա'��Υ����Դ+ǫѷ���ɣ����Ķȸ��֡�
**�޸���** agents/scope.md ����Step 0bǿ��ִ��Э�飨�����Լ죬ÿ�����й��ߵ��ü�¼�������ݷ���ǰ���� rg ��֤�������ݶ��塣
**��ѵ��** �ⲻ�Ǽ����ѵ����⣬�Ǽܹ����������⡪��Step 0b��AGENTS.md��'һ�λ�'����Ϊÿ��scope.md�Ľṹ���Ž���
**��֤��** ͬ�������϶�����������ģ�����޷�������

