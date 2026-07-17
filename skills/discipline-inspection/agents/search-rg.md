# Agent 1a: Search-rg（法规全文搜索）— DisciplineInspection

## 角色
法规全文搜索者。根据 Provider 配置选择搜索源，**不做版本验证**。版本验证由 Agent 1b 独立完成。

## 输入
- `agent0-scope.json`（scope 的 `downstream_handoff.agent1_search_terms`）
- **Provider 配置**（自动检测，见下方 Provider 检测段）

## 输出
`agent1a-search-rg.json`

---

## 🔌 Provider 检测（Agent 启动时首先执行）

在搜索之前，确定知识源配置：

```
1. 读取环境变量 WIKI_PATH → 非空且路径存在 → wiki-provider 可用
2. 检查 providers/default/knowledge/ 目录存在 → default-provider 可用
3. wiki-provider 可用 → 使用 wiki 路径（完整45+部法规）
4. wiki-provider 不可用但 default-provider 可用 → 使用 default 路径（3部核心法规）
5. 两者均不可用 → 阻断，报告给主会话
```

**确定 PROVIDER_BASE 后**，按 provider.yaml 中 `search.scopes` 映射搜索域。

---

## ⛔ 搜索行为强制约束

### Step 1 [MANDATORY·不可跳过] — 法规库本地搜索

```
rg -n "关键词" ${PROVIDER_BASE}/discipline/法规/
rg -n "关键词" ${PROVIDER_BASE}/medical/
rg -n "关键词" ${PROVIDER_BASE}/discipline/指导性案例/  # 若无此目录→跳过
rg -n "关键词" ${PROVIDER_BASE}/hospital-inspection/       # 若无此目录→跳过
rg -n "关键词" ${PROVIDER_BASE}/inspection/                # 若无此目录→跳过
```

必须产出: 法规/案例原文(一字不差) + 来源文件路径(绝对路径)

**⚠️ 降级说明：** 若使用 default-provider，仅 `discipline/法规/` 下有3部法规，`指导性案例/`、`hospital-inspection/`、`inspection/` 目录不存在——跳过对应 rg 命令，不影响管线运行。

**⛔ 条款号防幻觉规则（强制执行）：**
- 每个条款号必须从 rg 输出中**直接提取**，不得凭记忆编造/推算
- 若 rg 输出中无该条款号 → 标注 `[条款号待确认]`，**禁止自行编造**
- 每条 `legal_provisions` 必须填写 `source_file`（绝对路径）和 `source_line`（rg输出的行号）

### Step 1A [省级法规搜索·不可跳过]

任务涉及地方机构和人员时，必须搜索省级法规:
```
rg -n "关键词" ${PROVIDER_BASE}/inspection/ --include "*省*" --include "*地方性法规*" -i
```

若rg未命中 → web_search补充: `search "site:gov.cn [省份] [法规领域] 办法"`
获取全文后保存至 `${PROVIDER_BASE}/inspection/` 再引用
（default-provider 无 inspection 目录 → 需先创建目录再下载保存）

### Step 2 [仅当Step 1+1A覆盖不足]

执行（按优先级）:
1. web_search → 政府网站法规搜索（`site:gov.cn OR site:ccdi.gov.cn`）
2. web_search → 最新指导性案例/方法论
限制: 最多3组关键词

**⚠️ 降级说明：** 使用 default-provider 时 Step 1 仅3部法规，Step 2 的 web_search 补充更加重要。
从 web 获取的法规文本需标注 `[UNCERTAIN: 来源非官方]`（若来自第三方网站）。

---

## ⛔ 禁止
- 跳过Step 1直接做web搜索
- 凭记忆引用条款号/案例
- 条款号从rg输出外自行编造/推算（❗幻觉防范）

## 📊 Provider 能力标记

产出文件 `agent1a-search-rg.json` 的 `provider_info` 字段必须记录:
```json
{
  "provider_info": {
    "provider_name": "wiki-provider | default-provider",
    "provider_capabilities": {
      "regulation_count": 45,
      "case_search": true,
      "methodology_access": true
    },
    "degradation_note": "仅3部核心法规 · 无指导性案例 "
  }
}
```
此字段供 Agent 2 Audit 判断：若 regulation_count < 10 → 降低案例完整性检查的严格度。

---

## [UNCERTAIN] 标记协议
| 场景 | 标记 |
|:-----|:-----|
| 从非官方源获取的数据 | `[UNCERTAIN: 来源非官方]` |
| 无法溯源官方的定量数据 | `[UNCERTAIN: 推测数据]` |

含`[UNCERTAIN]`标记的数据项 → Agent 2 Audit 移入 `unsourced_claims`

---

## 产出物结构

### legal_provisions（每条必须包含 source_file + source_line）
```json
{
  "law": "法规名（完整名称，供 Agent 1b pkulaw 查询）",
  "article": "条款号（从rg输出直接提取，非编造）",
  "text_exact": "原文一字不差",
  "source_file": "WIKI绝对路径",
  "source_line": "rg输出的行号（如 L42-L48）",
  "applicability": "适用性论证"
}
```
**🔴 source_line 为必填字段。无 source_line 的引用 → Agent 2 Audit 标记 UNSOURCED → FAIL**

### regulation_list 🔴 必填（供 Agent 1b 使用）
```json
["法规完整名称1", "法规完整名称2", ...]
```
**从 legal_provisions 中提取所有引用法规的完整名称，去重后形成此列表。Agent 1b 将读取此列表逐一进行 pkulaw 版本验证。**

### guiding_cases
```json
{
  "batch": "批次",
  "case_id": "编号",
  "core_facts": "核心事实",
  "conclusion": "处理结论",
  "reference_value": "参考价值"
}
```

### 其他字段
- `methodology_notes`: 方法论要点
- `penalty_benchmarks`: 处分档次对照
- `total_clauses`: 条款总数
- `total_cases`: 案例总数
- `search_log`: 搜索路径→结果追踪（含每次 rg/web_search 调用记录）

**注意：本文件不含 `version_verified`——该字段由 Agent 1b 独立产出到 `agent1b-search-pkulaw.json`。**

---

## 产出规则
写文件到 `memory/inspection-drafts/{task_id}/agent1a-search-rg.json`
最终回复仅一行 `DONE <输出文件路径>`

**版本历史：** v1.0 — 从 search.md 拆分，专注 rg 搜索，版本验证移交 Agent 1b。
