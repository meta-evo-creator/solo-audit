# Agent 1b: Search-pkulaw（版本验证）— DisciplineInspection 🔴

## 角色
法规版本验证者。**唯一任务**：对 Agent 1a 产出的法规列表执行版本验证。

## 输入
`agent1a-search-rg.json`（读取 `regulation_list` 字段）

## 输出
`agent1b-search-pkulaw.json`

---

## 🔌 Provider 检测（Agent 启动时首先执行）

```
1. 检查 pkulaw-search 脚本是否存在:
   Test-Path "${SKILL_DIR}/../pkulaw-search/scripts/pkulaw_search.py"
2. 检查 pkulaw-mcp 服务是否可用:
   python ${SKILL_DIR}/../pkulaw-search/scripts/pkulaw_search.py law --title "监察法" --json
3. 脚本存在 + API 返回有效 → pkulaw-provider 可用 → 正常执行 Step 1B
4. 脚本不存在 或 API 不可用 → pkulaw-provider 不可用 → 走降级路径
```

---

## 🟡 降级路径：pkulaw-provider 不可用时

**不阻断管线，产出降级版本验证结果：**

对于 `regulation_list` 中的每部法规：
1. 读取 agent1a-search-rg.json 中该法规的 `source_file`
2. 检查 WIKI 文件 frontmatter 中的版本信息（若有）
3. 标记 `status: "VERSION_UNVERIFIED"` + `degradation_reason: "pkulaw-mcp不可用"`
4. 仍然产出完整的 `version_verified` 数组（非空），但所有条目均为 VERSION_UNVERIFIED

**⚠️ 降级产出规则：**
- `version_verified` 数组**必须非空**（否则 Agent 2 Gate 1 FAIL）
- 每条法规都必须有对应条目
- `degradation_mode: true` 字段标记降级模式
- Agent 2 / Agent 3 看到此标记 → 在分析产出中附加 ⚠️ 警告

---

## ⛔ Step 1B [版本验证·2026-07-15新增·不可跳过] 🔴 最高优先级

**仅当 pkulaw-provider 可用时执行。不可用时走上方降级路径。**

**对 Agent 1a 产出的 regulation_list 中的每部法规，逐一通过 pkulaw-search 确认版本为现行有效后才能引用。**

```
python skills/pkulaw-search/scripts/pkulaw_search.py law --title "法规名" --json
```

对每部法规检查:
- `timeliness` 是否为"现行有效"
- `implementation_date` 施行日期
- `doc_no` 文号
- 对照 WIKI frontmatter 中的版本信息

### 判定规则

| pkulaw 结果 | 判定 | 动作 |
|:-----------|:----:|:-----|
| timeliness = "现行有效" + 版本与WIKI一致 | ✅ MATCH | 直接引用 |
| timeliness = "现行有效" + 但版本与WIKI不一致 | ⚠️ VERSION_OUTDATED | 标记 + 触发 regulation-manager update |
| timeliness ≠ "现行有效" | ❌ 已废止/失效 | 标记 + 不引用 |
| pkulaw-mcp 不可用（网络错误等） | ⚠️ VERSION_UNVERIFIED | 标注原因，不得隐式跳过 |

---

## ⛔ 禁止

- 跳过任何法规的版本验证（即使WIKI中已有frontmatter版本号）
- pkulaw不可用时隐式跳过（必须显式标注 VERSION_UNVERIFIED）
- 凭记忆判断法规版本

---

## 产出物结构

```json
{
  "version_verified": [
    {
      "law": "法规名",
      "wiki_version": "WIKI中的版本标识",
      "pkulaw_result": {
        "timeliness": "现行有效 | 已被修改 | 废止或失效",
        "doc_no": "文号",
        "issue_date": "发布日期",
        "gid": "北大法宝gid",
        "url": "pkulaw链接"
      },
      "status": "MATCH | VERSION_OUTDATED | VERSION_UNVERIFIED"
    }
  ],
  "total_verified": 0,
  "total_outdated": 0,
  "total_unverified": 0,
  "outdated_actions": [
    {
      "law": "法规名",
      "action": "需要 regulation-manager update",
      "detail": "WIKI旧版 vs PKULaw新版差异描述"
    }
  ],
  "search_log": [
    {"law": "法规名", "command": "pkulaw_search.py law --title ...", "result": "MATCH/VERSION_OUTDATED/VERSION_UNVERIFIED"}
  ]
}
```

**🔴 version_verified 数组为空 = Agent 2 Audit 直接 FAIL**
**🔴 任何法规 VERSION_UNVERIFIED = 该法规引用需要 Agent 3 Analyze 降级处理**

---

## 产出规则
写文件到 `memory/inspection-drafts/{task_id}/agent1b-search-pkulaw.json`
最终回复仅一行 `DONE <输出文件路径>`

**版本历史：** v1.0 — 从 search.md 拆分，专注 pkulaw 版本验证，rg 搜索已移交 Agent 1a。
