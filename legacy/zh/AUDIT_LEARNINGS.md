# AUDIT_LEARNINGS.md — 审计层生效规则

> ⛔ 由 SOLO 事后审计器自动维护，不可手动编辑。
> 每次审计提案经石冰审批通过后，在此追加一条生效规则。

---

## 已废弃规则

| 规则ID | 生效日期 | 废弃日期 | 废弃原因 | 审批人 |
|:-------|:--------:|:--------:|:---------|:------:|
| **CHG-003** | 2026-05-24 | **2026-05-30** | checkpoint两步write→edit是cron故障根因。新设计移除checkpoint，审计改为cron run log | 石冰 |
| **CHG-005** | 2026-05-26 | **2026-05-30** | G4.5签名原为审计追踪，但签名写入触发了edit操作导致cron delivery阻塞 | 石冰 |
| **CHG-005a** | 2026-05-28 | **2026-05-30** | checkpoint中段迁移策略随checkpoint整体废弃 | 石冰 |

---

## 已生效规则

| 规则ID | 生效日期 | 来源审计 | 规则内容 | 适用范围 |
|:-------|:--------:|:---------|:---------|:---------|
| CHG-001 | 2026-05-23 | SOLO-AUDIT-20260523-001 | cron web_fetch 后必须检查 rawLength<500/反爬关键词/导航页脚，命中即切 babata-browser | 所有 cron 任务 |
| CHG-002 | 2026-05-23 | SOLO-AUDIT-20260523-001 | SOLO v2.0 三权制衡架构：审计层只读不写，只提提案不改SOP | 全系统 |
| CHG-004 | 2026-05-26 | P12 | ⛔ 修复标记「验证式」流程：任何修复标记为「已执行」前，必须先读取目标cron payload → 验证修改存在 → 写read证据到审计日志。禁止声明式标记。 | 审计层修复执行流程 |

---

## 审批通过 — 已执行（2026-05-26 22:15 石冰审批）

| 提案ID | 提出日期 | 优先级 | 问题 | 修复内容 | 验证方式 |
|:-------|:--------:|:------:|:-----|:---------|:--------:|
| **P10** | 05-25 | P0 | CHG-003合规不对称——修复仅覆盖1/3 cron任务。AI热点和AI医疗仍无checkpoint（day4复现） | 🔄 第二轮修复：Payload精简+签名嵌入报告正文+前置提醒+收口双重确保。cron get验证payload已更新 | 次日运行后检查checkpoint文件是否存在 |
| **P11** | 05-25 | P1 | AI热点+AI医疗 G3交付无确认（IMA code/WeCom message_id缺失） | G3交付记录嵌入报告footer和G4.5签名 | 次日报告末尾含ima_code/wecom_message_id字段 |
| **P12** | 05-25 | P2 | 修复标记审计——从声明式改为验证式（NP-002根因） | CHG-004已生效。本轮P10/P11修复执行时已采用验证式流程（cron get读payload确认修改存在） | 审计日志含read证据链 |
| **P13** | 05-26 | P0 | 🚨 AI医疗cron 05-26完全静默（无报告+无checkpoint） | 诊断：cron alive(enabled=true,lastRun=ok,delivered=true)但模型未写文件。修复：①Payload精简②签名嵌入报告③手动触发恢复(已入队) | 手动触发结果+次日09:00正常运行 |

### 历史（2026-05-24 22:35）

| 提案ID | 提出日期 | 问题 | 根因 | 修复内容 | 验证方式 |
|:-------|:--------:|:-----|:------|:---------|:---------|
| **P1** | 05-23 | 医院监督cron执行完毕但无checkpoint（G4.5签名缺失），审计层无法追踪执行结果 | cron payload末尾有G4.5 signature模板JSON但实际未写入文件 | 在cron payload末尾追加明确的文件写入指令：统计执行时长/duration_ms、失败URL列表/sources_failed、通过门禁/gates_passed → 写入 `memory/checkpoints/cron-hospital-YYYY-MM-DD.md` | 次日06:00运行后检查 `memory/checkpoints/cron-hospital-YYYY-MM-DD.md` 是否存在且含完整签名 |
| **P2** | 05-23 | AI热点cron同样缺失execution_signature JSON，与P1同构 | 同上，payload有模板但未指令写入 | 在cron payload末尾追加文件写入指令：`memory/checkpoints/cron-ai-hotspot-YYYY-MM-DD.md`（已有此文件但内容不完整）| 次日04:00运行后检查checkpoint含duration_ms/sources_failed/gates_passed |
| **P3** | 05-23 | cron G3门禁(IMA上传+PUSH)在交付自检中标记为PASS，但未验证实际执行结果 | 门禁依赖Agent自我声明，无IMP验证或外部调用确认 | 修改交付自检逻辑：G3改为「上传后等待code=0返回→标PASS」or「上传失败→标FAIL并写sources_failed」 | 次日运行后检查交付自检中G3的代码返回确认 |
| **P4** | 05-23 | MSF Draft阶段缺少G1.5交叉引用一致性校验：报告中引用的来源ID可能与scout/evidence_ledger中的ID不一致 | Draft Agent独立生成，无机制验证其引用的source ID在原始数据中确实存在 | MSF agent3-draft.md 新增验证步骤：Draft完成后，遍历报告中所有[Sxxx]引用 → 逐条验证在`evidence_ledger.json`中存在 → 不存在则标记`[UNSOURCED]`并记录report check | 下次MSF任务Draft阶段查看是否产生cross_reference_check.json |
| **P5** | 05-23 | FP-001修复（web_fetch反爬检测）是否已推送到医院监督cron？上次审计发现cron payload未同步 | FP-001属于core交叉影响规则，手动修复了cron但未验证推送完成 | 逐行对比cron payload vs MSF agent1-scout.md的反爬检测规则（4项检查），确认一致 | 已验证：cron payload已包含完整4项反爬检查（rawLength/反爬关键词/导航页脚/JS渲染），✅ 通过 |
| **P6** | 05-24 | solo-audit-daily-001 cron在24小时内触发3次（非预期的每天1次），怀疑重复执行 | 系统重启/cron调度器重启时重复触发已有job | ①确认p6-gp3检查的逻辑修复；②增加审计cron自身的去重保护（启动时检查上次运行时间<23h→跳过） | `cron runs solo-audit-daily-001` 查看最近3次运行，确认间隔>23h |
| **P7** | 05-24 | 所有cron的第三步（IMA上传+Wecom推送）依赖Agent自发执行→无强制验证机制→P1/P2/P3的根因 | G3/G4门禁为自主声明（自检自评），缺乏第三方IMP调用 | 统一模式：cron payload末尾G4.5签名 = ①文件写入(强制) → ②IMA上传 → ③Wecom推送 → ④写checkpoint | cron payload按此模式统一改写后验证 |
| **P8** | 05-24 | 🚨 医院监督checkpoint缺失已达28小时+ 100%复现 | P1未执行→cron无checkpoint写入 | 同P1修复 + 增加紧急验证标记 | 修复后次日06:00必须成功写入checkpoint |
| **P9** | 05-24 | 🚨 AI热点签名缺失已达28小时+ 100%复现 | P2未执行 | 同P2修复 + 增加紧急验证标记 | 修复后次日04:00必须成功写入 |
| **P10** | 05-25 | P0 | CHG-003合规不对称——修复仅推送了1/3 cron任务。AI热点和AI医疗cron仍无checkpoint（第三天→第四天复现） | 逐行验证AI热点+AI医疗cron payload是否包含：G4.5签名JSON写入memory/checkpoints/ + 写入后read验证 | ✅ P10已执行：2026-05-26 22:09 经石冰审批通过，cron payload已更新（`cron get`读payload验证G4.5签名+checkpoint写入+read验证三步齐全） |
| **P11** | 05-25 | P1 | AI热点+AI医疗G3交付无确认——报告末尾无IMA上传code/WeCom推送确认 | 参考医院监督cron合规模式：报告生成→IMA上传(记录code)→WeCom推送(记录确认)→写入G4.5签名 | ✅ P11已执行：2026-05-26 22:09 经石冰审批通过，cron payload已更新（交付步骤含ima_code/wecom_message_id/g3_delivery_complete字段） |
| **P12** | 05-25 | P2 | 修复标记审计——从「声明式」改为「验证式」。NP-002根因：修复在AUDIT_LEARNINGS.md标记已执行但未逐任务payload验证 | 任何修复标记为「已执行」前，必须先读取目标cron payload→验证修改存在→写read证据到审计日志 | ✅ P12已执行：2026-05-26 22:09 经石冰审批通过，CHG-004规则已写入

---

## 已执行修复项（2026-05-24 22:53）

| 编号 | 问题 | 修复 |
|:----:|:-----|:------|
| **BUG-001** | 审计cron路径 `~` 不解析→write失败 | ✅ 全部路径改为绝对路径 `C:\Users\shibi\.openclaw\workspace\` |
| **BUG-002** | 审计cron不写G4.5 checkpoint | ✅ 追加CHG-003强制收口+read验证 |
| **BUG-003** | 提案审后无限等待 | ✅ 新增Step⑤自愈检查：自动读石冰确认消息→标记为已批准 |
| **P1-P9** | 9条提案全部待审批 | ✅ 石冰已确认全部通过，AUDIT_LEARNINGS.md 已归档为「已生效」 |
| **CHG-003** | 所有cron checkpoint强制规则 | ✅ 已写入AUDIT_LEARNINGS.md，所有cron启动时自动加载 |
| **CHG-004** | 修复标记验证式流程 | ✅ 已写入AUDIT_LEARNINGS.md，P10执行时已按此流程验证（cron get读payload确认修改存在） |
| **P10** | CHG-003覆盖率修复 | ✅ AI热点cron + AI医疗cron payload已更新，含G4.5签名+checkpoint写入+read验证（cron get验证payload存在） |
| **P11** | G3交付自动化 | ✅ AI热点cron + AI医疗cron payload已更新，交付步骤含ima_code/wecom_message_id/g3_delivery_complete追踪 |
| **P12** | 修复标记审计 | ✅ CHG-004规则已生效，P10/P11执行时已采用验证式流程 |

---

---

## 审批通过 — 已执行（2026-05-28 22:29 石冰审批）

| 提案ID | 提出日期 | 优先级 | 问题 | 修复内容 | 验证方式 |
|:-------|:--------:|:------:|:-----|:---------|:--------:|
| **P14** | 05-28 | P0 | P10修复无效——AI热点+AI医疗继续缺失checkpoint。CHG-005覆盖率33% | ✅ CHG-005a已生效：checkpoint写入迁移至报告中段+验证cron update实际生效 | TODO: 下次运行检查checkpoint存在 |
| **P15** | 05-28 | P1 | AI热点G3交付持续pending | ✅ 与P14合并修复 | 同上 |

---

## 审批通过 — 已执行（2026-05-31 09:18 石冰审批）

| 提案ID | 提出日期 | 优先级 | 问题 | 修复内容 |
|:-------|:--------:|:------:|:-----|:---------|
| **P16** | 05-31 | P2 | cron-deliver.cjs清洗词表缺G3/G0/G1/G2/门禁等→报告含G3元数据上传IMA | 清洗数组W追加 `'G3','G0','G1','G2','门禁','交付门禁','quality_gate'`，已有G4.5对称覆盖 |

## 已生效规则

| 规则ID | 生效日期 | 来源审计 | 规则内容 | 适用范围 |
|:-------|:--------:|:---------|:---------|:---------|
| CHG-006 | 2026-05-31 | SOLO-AUDIT-20260531-001 | cron-deliver.cjs清洗词表全量覆盖G0-G4门禁关键词（G4.5已有，补全G3/G0/G1/G2/门禁/quality_gate） | cron-deliver.cjs |

## 审批中提案

（无待审批提案）

---

### 历史审批（2026-05-26 22:15 石冰审批）

> 2026-05-26 22:15 石冰消息：「复现的问题还不处理，我都审批通过，赶紧都修复」— P10/P11/P12/P13全部审批通过并已执行。

### 历史审批（2026-05-24 22:35 石冰审批）

> P1-P9全部通过，已归档为「已生效」。

> 最后更新：2026-05-29 22:00 CST

### P14/P15 验证结果（2026-05-29）

| 提案ID | 提出日期 | 验证日期 | 结果 | 证据 |
|:-------|:--------:|:--------:|:----:|:-----|
| **P14** | 05-28 | 05-29 | ✅ **已验证有效** | AI热点 checkpoint 05-29 PRESENT (1169B, full signature) |
| **P15** | 05-28 | 05-29 | ✅ **已验证有效** | AI医疗 checkpoint 05-29 PRESENT (1475B, full JSON, ima_code=0) |

**BREAKTHROUGH:** CHG-003覆盖率 0.33(05-28) → 1.00(05-29)。首次实现3/3 cron checkpoint合规。CHG-005a (checkpoint迁移至报告中段) 打破NP-004假说。

---

## CHG 提案格式（version 2026-05-31）

所有 agent 启动时 `read AUDIT_LEARNINGS.md`，需了解两类提案的格式。

### 类型一：行为修正提案（solo-audit 产出）

```markdown
## CHG-XXX: [问题描述]

**来源:** SOLO-AUDIT-YYYYMMDD-NNN
**严重度:** P0 | P1 | P2
**问题:** [一句话]
**根因:** [为什么发生]
**修复:** [怎么改·消路径优先]
**范围:** [影响哪些文件/cron]
**验证:** [怎么验证修好了]
```

### 类型二：文本优化提案（SkillOpt 离线训练产出）

> 适用：DR / MSF / solo-audit 三个技能的 SKILL.md 自动优化。DI 除外。

```markdown
## CHG-XXX: [技能名] [agent名] 文本优化

**来源:** SkillOpt 离线训练
**验证结果:** best_skill.md Review 总分 XX/100 vs 原skill XX/100
**严重度:** P2（仅当得分≥原版且石冰审批才生效）

**Diff:**
<<< OLD
原 SKILL.md 对应段落
>>>
<<< NEW
优化后段落
>>>

**验证证据:**
- old win-rate: XX/100 (N runs)
- new win-rate: XX/100 (N runs)

**石冰审批:** [ ] 通过 [ ] 拒绝
```

两类提案的共同底线：**石冰审批通过前，任何修改不生效。铁律④权责。**
