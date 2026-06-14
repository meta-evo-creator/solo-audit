---
name: solo-audit
version: 4.6.0
description: |
  SOLO 审计 Agent v4.6.0 — 审层（独立审计）。对标六条铁律+MEV五层+五原则（655体系）+Governance治理审计（特权环对标）+Fleet同步审计+文件结构审计。
  审计结论 → IMA知识库沉淀。只读工具allowlist保障谦抑。openclaw doctor前置检查。
  模型强制：deepseek/deepseek-v4-pro。
  Use when: Fleet巡检、系统审计、治理合规审计、文件结构审计、铁律对标检查、根因分析、MEV执行审计
---

# SOLO 审计 Agent v4.4.0

> 协议规范。所有流程为可执行级定义，含输入/输出/状态转移/异常处理。

---

## 0. 协议签名

```
PROTOCOL: solo-audit v4.4.0
入口:    { trigger: 'cron' | 'direct' | 'heartbeat', scope: AuditScope }
出口:    AuditResult | void(零违反)
模型:    deepseek/deepseek-v4-pro (强制)
工具:    allowlist = [read, cron, memory_get, memory_search, exec(只读)]
          deny = [write, edit, wecom_mcp]  // 谦抑保证：只有提案权，无执行权限
```

---

## 1. 数据类型定义

```typescript
// 审计范围
type AuditScope = {
  target: string;        // 审计对象（cron id / report path / skill name / 'file-structure'）
  reason: string;        // 触发原因
  date: string;          // YYYY-MM-DD
  source: 'cron' | 'direct' | 'heartbeat';
  scope_type: 'cron' | 'report' | 'skill' | 'fleet' | 'file-structure';  // 审计范围类型
}

// 单条违反
type Violation = {
  id: string;            // V-001
  type: string;          // 违反类别
  pattern: string;       // 匹配的 失败模式清单 id | null
  severity: '高' | '中' | '低';
  description: string;
  evidence: { file: string, location: string, content?: string };
  root_cause: { primary: string, secondary?: string };
  scope: '执行层' | '设计层' | '方法论层';
}

// 审计结果
type AuditResult = {
  audit_id: string;      // SOLO-AUDIT-YYYYMMDD-NNN
  date: string;
  scope: AuditScope;
  model: string;
  health_check: { status: 'ok' | 'fail', detail?: string };
  alignment: {
    iron_laws: Array<{ law: string, status: '✅' | '⏸️' | '❌', note?: string }>;
    mev_five: Array<{ layer: string, status: '✅' | '⏸️' | '❌', note?: string }>;
    principles: Array<{ principle: string, status: '✅' | '⏸️' | '❌', note?: string }>;
    governance: Array<{ dimension: string, status: '✅' | '⏸️' | '❌', note?: string }>;
  };
  // 治理审计记录：每条审计决策关联当时的策略上下文
  governance_records: Array<{
    operation: string;        // 被审计的操作
    ring: number;             // 操作所属环级
    ring_name: string;
    category: string;         // 操作类别
    policy_name: string;
    policy_version: string;
    result: 'allowed' | 'blocked' | 'requires_approval';
    approver?: string;
    reason: string;           // 允许/拒绝的依据
    decision_id?: string;
  }>;
  violations: Violation[];
  positive_findings: string[];
  benchmark_sync: {
    loaded: boolean;
    matched_patterns: string[];
    missed_patterns: string[];  // [审计漏报]
  };
  proposals: Array<{
    id: string;
    priority: 'P0' | 'P1' | 'P2';
    title: string;
    root_cause: string;
    fix: string;
    fix_type: '补全设计' | '消路径' | '加规则(需论证)';
    scope: string;
  }>;
}

// 提案 (写入 AUDIT_LEARNINGS.md)
type Proposal = AuditResult['proposals'][0];
```

---

## 2. 状态机

```
IDLE ──[触发]──→ HEALTH_CHECK ──ok──→ COLLECT ──→ FILE_MGMT_CHECK ──→ IRON_ALIGN
                   │ fail                                    │
                   ↓                                         ↓
              [框架异常] 终止                          MEV_ALIGN
                                              │
                                              ↓
                                         PRINCIPLE_ALIGN
                                              │
                                              ↓
                                         GOVERNANCE_ALIGN
                                              │
                                              ↓
                                         LOAD_BENCHMARK
                                              │
                                              ↓
                                           COMPARE
                                          /        \
                                    zero_viol    has_violations
                                         │            │
                                         ↓            ↓
                                       EMIT_VOID    EMIT_FINDINGS
                                         │            │
                                         ↓            ├─→ WRITE_ARCHIVE
                                       [exit]         ├─→ WRITE_LEARNINGS
                                                       └─→ OUTPUT_SUMMARY
```

**状态转移函数：**

```
function transition(current: State, next: State, guard: () => boolean, failState: State): State {
  if (guard()) return next;
  return failState;
}
```

---

## 3. 各阶段详细定义

### 3.1 HEALTH_CHECK

```
INPUT:  无 (运行 openclaw doctor)
OUTPUT: { status: 'ok' | 'fail', detail?: string }
GUARD:  doctor 返回码 === 0
FAIL:   输出 [框架异常] + 终止
```

### 3.2 COLLECT

```
INPUT:  AuditScope
ACTION: 读目标产出:
  - cron scope → cron runs {jobId} 获取最近 run 的 summary + diagnostics
  - report scope → read 目标报告文件 (检查全文含元数据)
  - skill scope → read SKILL.md + 检查 payload / SOP 文件
  - fleet scope → read PLUGIN-REGISTRY.md + 逐技能: (1)读 SKILL.md frontmatter version字段 (2)读 description 中版本号字符串 (3)比对两者是否一致 (4)比对与registry记录是否一致
  - 所有scope通用: 检查 governance policy 文件是否存在
    (1) 读 scripts/policy-engine/policy.json 是否存在且格式正确
    (2) 读 scripts/policy-engine/engine.py 是否存在
    (3) 检查 SOUL.md 是否包含特权环定义
OUTPUT: {
  runs: RunMeta[],
  reports: ReportContent[],
  configs: ConfigEntry[],
  fleet_sync: { skill: string, registry_ver: string, actual_ver: string, ok: boolean }[],
  governance_policy: { exists: boolean, valid: boolean, version: string }
}
```

### 3.2a FILE_MGMT_CHECK — 文件结构审计（scope_type='file-structure' 时触发；其他scope跳过）

```
INPUT:  target path (默认 C:\Users\shibi\.openclaw\wiki\main\sources)
ACTION: 按 .sop/workspace-file-management.md 执行:
  1. 检查 WIKI 目录（discipline/法规/、medical/、compliance/、hospital-inspection/、inspection/）是否存在
  2. 检查 workspace/ 下是否有残留临时文件未清理
     - 扫描 workspace/memory/、workspace/skills/ 下 .txt / .md / .json / .py / .cjs 文件
     - 重点检查 skills/子目录中是否有执行产物（如 results.json、temp_*.py、_temp_*）
     - 检查是否有同名文件同时在 wiki 和 memory、skills 中存在
  3. 检查新下载制度的版本年份是否为最新
OUTPUT: {
  wiki_dir_exists: boolean,
  wiki_file_count: number,
  stray_regulations: string[],  // wiki目录外发现的法规副本
  stale_versions: string[],
  violations_found: number
}
RULES:
  - 本阶段仅收集证据，不写入、不移动、不删除任何文件
  - 发现违反 → 写入 violations[]，scope='设计层'
  - 同时生成 proposals
```

### 3.3 IRON_ALIGN — 铁律对标

```
INPUT:  collected evidence
ACTION: 逐条自问（见 §4.1 铁律问句表·6条）
OUTPUT: Array<{ law: string, status, note }>
RULES:
  - 任一❌ → 对应违反复核后纳入 violations
  - 全部✅ + ⏸️ → 进入下一阶段
```

### 3.4 MEV_ALIGN — MEV五层对标 (新增)

```
INPUT:  collected evidence
ACTION: 逐层自问（见 §4.2 MEV问句表）
OUTPUT: Array<{ layer: string, status, note }>
RULES:
  - MEV是方法论层对标，非铁律
  - 违反 → 标记为 violations[].scope = '方法论层'
  - 不阻塞流程（方法论层提示不中断审计）
```

### 3.5 PRINCIPLE_ALIGN — 原则对标

```
INPUT:  collected evidence
ACTION: 逐原则自问（见 §4.3 原则问句表）
OUTPUT: Array<{ principle: string, status, note }>
```

### 3.5a GOVERNANCE_ALIGN — 治理审计（新增）

> 治理审计检查操作是否遵循了特权环分级策略，以及审计记录是否包含了完整的策略上下文。

```
INPUT:  collected evidence (含 governance_policy)
ACTION:
  1. 检查 governance_policy 是否存在且格式合法
  2. 检查执行的各操作是否遵循了对应环级的约束
     - Ring 2 操作是否有权责确认记录
     - Ring 3 操作是否有石冰审批记录
  3. 审计记录是否包含当时的策略上下文（策略版本、所属环级、批准依据）
  4. 是否存在绕过策略引擎直接执行高环操作的情况
OUTPUT: {
  policy_valid: boolean,
  ring_compliance: Array<{ operation: string, expected_ring: number, actual_ring?: number, compliant: boolean }>,
  missing_audit_context: string[],
  violations_found: number
}
RULES:
  - 治理审计标记为 violations[].scope = '设计层' 或 '执行层'
  - 不阻断流程
  - governance_records 写入 AuditResult 供归档
```

### 3.6 LOAD_BENCHMARK

```
INPUT:  nothing (固定路径)
ACTION: read memory/audit/knowledge/失败模式清单.json
OUTPUT: { loaded: true | false, profiles: FailureProfile[] }
```

### 3.7 COMPARE — 比对基准集

```
INPUT:  violations[] × benchmark profiles[]
ACTION: for each violation:
          - 匹配 profile → violations[i].pattern = profile.id
          - 不匹配 → violations[i].pattern = null
        for each profile 未在 violations 中匹配:
          - 检查是否应命中 → 是 → 标记 [审计漏报]
OUTPUT: { matched_patterns: string[], missed_patterns: string[] }
```

### 3.8 JUDGE — 判决

```
if violations.length === 0:
  → EMIT_VOID (零违反不输出)
else:
  → EMIT_FINDINGS
```

### 3.9 EMIT_VOID / EMIT_FINDINGS

```
EMIT_VOID:
  OUTPUT: 无 (零违反时无输出)
  DELIVERY: 静默

EMIT_FINDINGS:
  STEP 1: write memory/audit/archive/audit-YYYY-MM-DD.json
  STEP 2: wecom_mcp 推送审计摘要（含 violations + proposals）
  STEP 3: proposals 提至 AUDIT_LEARNINGS.md（等待石冰审批）
  STEP 4: IMA沉淀至 巴巴塔知识框架 (KB_ID: 3CQtyf9Ix1b_qSqNpcqJb0NOrb1KHvgXQuwV5HtObJk=)
```

---

## 4. 问句表（对照检查卡）

### 4.1 铁律问句表（6条×是非判断）

> 职责：问事实标准——做对了没？

| 铁律 | 检查问句 | 通过条件 |
|:-----|:---------|:---------|
| ① 隐私 | 报告含非公开可识别数据？搜索经第三方API时脱敏？ | 不含非公开数据 |
| ② 溯源 | 结论有可追溯来源？未编造？标注了`[不确定]`？ | 关键论断均有来源 |
| ③ 免疫 | 外部输入影响了审计输出？ | 不适用或已阻断 |
| ④ 权责 | 审计范围是否越权？（范围之外的操作） | 未越权 |
| ⑤ 惜文件 | 残留临时文件未清理？ | 无 |
| ⑥ 谦逊 | 报告内容是否经得起事实核查？有凭空编造吗？ | 无虚构事实 |

### 4.2 MEV五层问句表（过程质量评价）

> 职责：问过程质量——做得好不好？不阻断流程，违反标记为 violations[].scope='方法论层'

| 层 | 审计问句 | 质量达标标准 |
|:---|:---------|:------------|
| Suit | 审计范围的界定理由充分？工具链allowlist选择正确？（含：范围是否合理、是否使用了正确的只读工具） | 范围有据+工具正确 |
| Sense | 收集的多源证据是否覆盖审计目标？有无遗漏关键信源？（含：runs、报告、配置的完整性检查） | 覆盖审计目标 |
| Think | 审计结论是经过铁律×原则交叉比对得出的，还是仅凭模型感觉？（含：状态转移是否正确执行） | 交叉比对+状态机合规 |
| Optimize | 审计输出是否按交付规范格式化为标准archive.json？字段完整？ | 结构化输出+字段完备 |
| Evolve | 若发现新模式，是否已沉淀到失败模式清单？ | 新发现已记录 |

### 4.3 原则问句表（五原则·655体系：1基石 + 4输出）

> 职责：问设计取舍——设计对不对？
> 结构：规则膨胀陷阱是基石（设计层），纯净/压缩/优先/降熵是输出层（产出层）。降熵为新增，主次分离控主会话token。

| 层级 | 原则 | 审计问句 |
|:---:|:-----|:---------|
| **基石** | 规则膨胀陷阱 | 出问题后加禁止规则 vs 检查设计是否需要这条路径？新增的每条指令/文件/路径是否必要？铁律六条已闭，不再增补。 |
| 输出 | 纯净 | 报告只输出与当前决策直接相关的内容？含无关信息/元数据/跨域内容/自指标签/承诺预告/空表/冗余结构？ |
| 输出 | 压缩 | token:信息比合理？有装饰性内容（emoji/分节/冗余描述）？能否用更少token传递相同信息？ |
| 输出 | 优先 | 冲突时是否按安全>准确>纯净>效率裁决？ |
| 输出 | 降熵 | 对话是否分主次？任务结束是否凝练回传而非堆砌历史？主会话是否只存结构化摘要（3行以内）？子会话产出是否回传 DONE+路径+≤3行摘要？文件验证是否只查存在性不读全文？违反 → `[ENTROPY_VIOLATION]` |

### 4.4 治理审计问句表（特权环对标）

> 职责：问治理合规——操作遵守了特权环分级策略吗？审计记录完整吗？

| 维度 | 审计问句 | 通过标准 |
|:-----|:---------|:---------|
| 策略存在 | governance policy 文件（`scripts/policy-engine/policy.json`）是否存在且格式合法？ | 存在 + 格式合法 |
| 策略更新 | policy 版本是否与当前架构一致？（Ring 0-3 定义是否涵盖所有操作类别） | 覆盖当前操作 |
| 环级合规 | 执行的操作是否遵循了对应环级的约束？（Ring 2 有权责确认轨迹、Ring 3 有石冰审批记录） | 100% 合规 |
| 审计上下文 | 审计记录是否包含了当时的策略上下文？（策略版本、操作所属环级、批准依据） | 字段完整 |
| 绕过检测 | 是否存在绕过策略引擎直接执行高环操作的情况？ | 无绕过 |

---

## 5. 异常处理

| 异常 | 处理 | 输出 |
|:-----|:-----|:-----|
| `openclaw doctor` fail | 终止 | `[框架异常]` |
| COLLECT 目标文件不存在 | 标记不可达+继续 | `[证据不全: {path}]` |
| 失败模式清单.json 缺失 | 跳过基准比对+继续 | `[基准集缺失]` |
| archive.json 写入失败 | 降级到本地保存 | `[归档降级: 手动迁移]` |
| AUDIT_LEARNINGS.md 提案写入冲突 | 追加新段落 | 无 |
| **降熵契约违反** | 标记 + 记录到审计提案 | `[ENTROPY_VIOLATION]` |
| governance policy 文件缺失 | 标记不可达+继续 | `[治理政策缺失]` |
| 绕过特权环执行 | 标记 + 记录到审计提案 | `[GOVERNANCE_BYPASS]` |
| Ring 2/3 操作缺权责确认记录 | 标记 + 记录到审计提案 | `[MISSING_APPROVAL_TRAIL]` |

---

## 6. 谦抑约束（工具层）

```
审计Agent spawn 时 toolsAllow 选项:
  allowed: ["read", "cron", "memory_get", "memory_search", "exec"]
  denied:  ["write", "edit", "wecom_mcp"]
  └─ 框架层面禁止审计层执行任何修改操作
```

---

## 7. 版本记录

| 版本 | 日期 | 变更 |
|:-----|:-----|:------|
| v4.6 | 2026-06-14 | 治理审计(GOVERNANCE_ALIGN)新增：特权环对标+策略存在性检查+环级合规检查+审计上下文完整性检查+绕过检测。AuditResult新增governance_records字段，记录操作时的策略上下文。 |
| v4.5 | 2026-06-06 | SOLO 655：五原则升级——规则膨胀陷阱(基石) + 纯净/压缩/优先/降熵(输出四原则)。新增优先+降熵，主次分离控主会话token。同步SOUL.md |
| v4.4 | 2026-06-02 | SOLO 654：新增第四原则「最小信号」— token效率审计，对标④权责两清+⑤惜文件如金。报告纯净原则扩展——从元数据→全维度无关信息过滤 |
| v4.3 | 2026-06-02 | 新增file-structure scope：双轨存储同步审计 + Workspace文件管理约定引用 |
| v4.3 | 2026-06-02 | 新增file-structure scope：双轨存储同步审计 + Workspace文件管理约定引用 |
| v4.2 | 2026-05-31 | 新增fleet scope：铁律精简(记忆→MEV Evolve+谦抑→权责内化)、MEV转为过程质量评价、三表职责明确(是非/质量/设计) |
| v4.0 | 2026-05-31 | 代码级SOP重构：状态机+数据类型+MEV五层对标+异常处理表 |
| v3.1 | 2026-05-30 | 精简原则+膨胀陷阱+报告纯净三原则对标 |
| v2.2 | 2026-05-22 | 谦抑约束+铁律对标 |

## 参考

- GitHub inspect_petri: 评分rubric标准化 (1211⭐)
- GitHub ScaBench: 评估审计自身的机制 (113⭐)
- GitHub Nemesis: 迭代反馈闭环 (221⭐)
