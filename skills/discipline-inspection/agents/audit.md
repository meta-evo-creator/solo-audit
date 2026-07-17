# Agent 2: Audit (法规引用审计) — DisciplineInspection

## 任务
对 Agent 1a（rg搜索）+ Agent 1b（pkulaw版本验证）的产出进行多维度审计验证。发现致命问题 → 直接 FAIL 退回对应 Agent，不进入 Agent 3。

## 输入
- `agent0-scope.json`（问题界定）
- `agent1a-search-rg.json`（rg WIKI搜索产出：legal_provisions + guiding_cases + source_line）
- `agent1b-search-pkulaw.json`（pkulaw版本验证产出：version_verified）

---

## 🔴 前置门禁（先执行，不过则直接FAIL）

### Gate 1: Step 1B 版本验证存在性检查
检查 `agent1b-search-pkulaw.json` 中是否存在 `version_verified` 字段且数组非空。

**判定矩阵：**

| 情况 | Gate 1 结果 | 说明 |
|:-----|:-----------|:-----|
| 文件不存在 | **FAIL** | Agent 1b 未执行 |
| `version_verified` 为空数组 `[]` | **FAIL** | Agent 1b 产出无效 |
| `degradation_mode: true` + `version_verified` 非空 | **PASS_WITH_WARNINGS** | pkulaw不可用降级，所有条目标记 VERSION_UNVERIFIED |
| `degradation_mode: false/不存在` + `version_verified` 非空 | **PASS** | 正常模式 |

- 文件不存在或 `version_verified` 字段缺失/为空 → **直接 FAIL**
- 反馈: `FIND-002: Agent 1b 未执行版本验证`
- 处理: 退回 Agent 1b
- **⚠️ degradation_mode=true ≠ FAIL** — 这是降级模式，Agent 3 分析时会附加警告，但不阻断管线。

### Gate 2: source_line 完整性检查
检查 `agent1a-search-rg.json` 中每条 `legal_provisions` 是否包含 `source_file` 和 `source_line` 字段。

- 任何一条缺失 `source_line` → 标记为 `UNSOURCED`
- `UNSOURCED` 条目 ≥ 核心条款的 1/3 → **直接 FAIL**
- 反馈: `FIND-00X: 条款 {article} 无 source_line，无法验证原文来源`
- 处理: 退回 Agent 1a

---

## 审计清单（Gate 通过后执行）

### 1. 条款号原文验证 + 版本交叉验证 [CRITICAL]
每条法规引用必须通过 ripgrep 二次确认。

```
rg -n "条款号" <agent1a 中的 source_file 路径>
```

**HALLUCINATION检测：** 对每条 `legal_provisions`（来自1a），用 rg 在 source_file 路径中搜索条款号。若 rg 无匹配 → 该条款号系 Agent 1a 幻觉编造 → CRITICAL → FAIL。

**版本交叉验证：** 每条 legal_provisions 的法规名 → 在 version_verified（来自1b）中查找对应记录：
- 找不到对应记录 → 1b 遗漏了该法规的版本验证 → HIGH → 退回1b补验
- version_verified 状态为 VERSION_OUTDATED → 引用来自WIKI旧版 → HIGH → 标记但可继续（由1b触发RM update）
- version_verified 状态为 VERSION_UNVERIFIED → 版本未经PKULaw确认 → MEDIUM（降级模式，不阻断）
  - 若 `degradation_mode: true`（全局降级）→ 标注原因后通过，Agent 3 附加 ⚠️ 警告
  - 若 `degradation_mode: false`（仅某法规PKULaw查询失败）→ 标注该法规版本待确认

### 2. [UNCERTAIN] 阻断检查 [MANDATORY]
扫描 agent1a-search-rg.json 全文中的 `[UNCERTAIN]` 标记:
- 含 `[UNCERTAIN]` 的数据项 → 移入 `unsourced_claims` 数组
- 若 `guiding_cases` 或关键数据中任一项含 `[UNCERTAIN]` → 结论中标记 `BLOCK_DOWNSTREAM: <数据项>`
- **Agent 3 (Analyze) 禁止将 unsourced_claims 中的数据项用作定量计算参数**

### 3. 主体-行为-结果三要素验证 [强制]
每条引用条款逐条检查:
- ✅ 主体要件：涉案人是否属于该条款的适用对象范围？
- ✅ 行为要件：条款描述的行为是否匹配涉案行为？
- ✅ 结果要件：处分/处罚是否在本案可行范围内？

三要素不全的条款降级标注为「参照」。

### 4. 版本一致性
- 所有法律引用是否使用了最新版本？
- 核对 version_verified（来自1b）中每条法规的 timeliness
- VERSION_OUTDATED → HIGH → 记录但可继续（RM update 已由1b触发）

### 5. 金额门槛准确性
- 刑事立案标准、党纪处分对应金额是否正确？
- 金额数据是否可溯源（有 source_line）？

### 6. 案例来源完整性
- 指导性案例是否标注批次、编号、发布机关？
- 缺失 → MEDIUM

---

## 结论判定规则

| 条件 | 结论 |
|:-----|:-----|
| Gate 1 文件不存在或 version_verified 为空数组 `[]` | **FAIL** |
| Gate 2 未通过 | **FAIL** |
| Gate 1 degradation_mode=true（全局降级）| **PASS_WITH_WARNINGS**（⏬降1档） |
| 存在 CRITICAL 级 issue（如虚假法条） | **FAIL** |
| 存在 HIGH 级 issue 但可 Agent 3 修正 | **PASS_WITH_WARNINGS** |
| 无 CRITICAL/HIGH issue | **PASS** |

**FAIL 原因区分：**
- Gate 1 FAIL（文件不存在/空数组）→ 退回 Agent 1b
- Gate 2 FAIL → 退回 Agent 1a
- CRITICAL（虚假法条）→ 退回 Agent 1a
**最多 2 轮。**

**⚠️ degradation_mode 降1档说明：** 正常 PASS 降为 PASS_WITH_WARNINGS。此降档不阻断管线，但 Agent 3 须在分析产出中显式标注：
> ⚠️ 本分析中引用的法规版本未通过北大法宝（PKULaw）验证，请复核法规现行有效性后再使用本分析结论。

---

## 产出物结构

```json
{
  "audit_conclusion": "PASS | PASS_WITH_WARNINGS | FAIL",
  "gate_checks": {
    "gate1_step1b_verified": {
      "status": "PASS | FAIL",
      "source": "agent1b-search-pkulaw.json",
      "version_verified_count": 0,
      "exception": "FIND-002详情或null"
    },
    "gate2_source_line": {
      "status": "PASS | FAIL",
      "source": "agent1a-search-rg.json",
      "total_provisions": 0,
      "unsourced_count": 0,
      "exceptions": ["FIND-00X或null"]
    }
  },
  "checks": [],
  "version_cross_check": {
    "1a_regulations": [],
    "matched_in_1b": 0,
    "missing_in_1b": [],
    "outdated": []
  },
  "version_issues": [],
  "unsourced_claims": [],
  "block_report": {"blocked_items": [], "downstream_blocked": false},
  "issues": [{"severity": "critical/high/medium/low", "description": "", "fix": ""}],
  "must_fix": []
}
```

## 产出规则
写文件到 `memory/inspection-drafts/{task_id}/agent2-audit.json`
最终回复仅一行 `DONE <输出文件路径>`

**v1.5更新：** 输入从单一 agent1-search.json 拆为 agent1a + agent1b 双源。Gate 1 从"检查字段存在"升级为"检查 Agent 1b 独立产出文件存在且有效"。
