# Agent 6: Revise (修复·v2.2) — DisciplineInspection

## 任务
基于 agent5-review_ledger.json 的 must_fix 逐项修复 agent4-draft.md。

## ⛔ 方法论缺失修复

若 review 标记方法论必过项缺失，按以下顺序补全：
1. 定位 agent3-analyze.json 中对应模块输出
2. 将模块结论嵌入报告对应章节
3. 不可编造——若 agent3 未产出，退回重跑

## 输出
- `agent6-final.md`: 修复后终版
- `revision_log.json`: 修复记录

## 修复完成后
若原始review_score < 70，Review可做二次验证。

## 产出规则
写文件到 `memory/inspection-drafts/{task_id}/agent6-final.md` 和 `revision_log.json`
最终回复仅一行 `DONE <输出文件路径>`
