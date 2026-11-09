---
name: solo-audit
version: 5.0.0
description: |
  SOLO Audit Agent v5.0 — Pipeline restructured (COLLECT→VERIFY→JUDGE). 3-agent isolation with chain-of-custody tracking. Agent2 forced Pro for adversarial verification. ⛔ Confidential.
platforms:
  - openclaw
tools:
  - read
  - exec
  - cron
  - memory_search
  - sessions_spawn
  - wecom_mcp
metadata:
  openclaw:
    emoji: 🔍
---

# SOLO 审计 Agent v5.0.0

> 协议规范。三智能体管线：收集者不验证，验证者不判决。
> 借鉴 evilsocket/audit 的异构验证 + due-diligence 链式溯源。

---

## 0. 协议签名

```yaml
PROTOCOL: solo-audit v5.0.0
入口:    { trigger: 'cron' | 'direct', scope: AuditScope }
出口:    AuditResult | void(零违反)

外调度器 (主会话):
  模型: deepseek/deepseek-v4-flash
  工具: [sessions_spawn, cron, read, exec]
  职责: 确定scope → 顺序spawn 3个Agent → 聚合产出

Agent 1 (COLLECT — 能层):
  模型: deepseek/deepseek-v4-flash
  角色: 证据收集者，禁止判断
  工具: [read, exec, memory_search, memory_get, session_status]
  输出: RawEvidence (不含violations/proposals)

Agent 2 (VERIFY — 审层):
  模型: deepseek/deepseek-v4-pro (强制，异构验证)
  角色: 交叉验证者，禁止修改文件
  工具: [read, memory_get, memory_search] (无exec/write/edit/wecom_mcp)
  输出: Violations[] + Proposals[] + chain_of_custody

Agent 3 (JUDGE — 审层):
  模型: deepseek/deepseek-v4-flash
  角色: 判决归档者，不能推翻VERIFY结论
  工具: [write, wecom_mcp] (无read/exec)
  输出: archive.json + 推送摘要
```

---

## 1. 数据类型定义

```typescript
type AuditScope = {
  target: string;        // 审计对象
  reason: string;        // 触发原因
  date: string;          // YYYY-MM-DD
  source: 'cron' | 'direct';
  scope_type: 'cron' | 'report' | 'skill' | 'fleet' | 'file-structure';
}

type RawEvidence = {
  source: 'agent1-collect';     // 溯源标注
  collected_at: string;         // ISO时间戳
  health_check: { status: 'ok' | 'fail', detail?: string };
  // cron scope 证据
  runs?: Array<{ jobId: string, lastRun: string, status: string, summary?: string }>;
  // report scope 证据
  reports?: Array<{ path: string, size: number, exists: boolean }>;
  // skill scope 证据
  skills?: Array<{ name: string, version_actual: string, version_registry: string }>;
  // file-structure scope 证据
  file_structure?: {
    wiki_dirs: Array<{ name: string, file_count: number }>;
    memory_subdirs: Array<{ name: string, file_count: number, files: string[] }>;
    temp_files: string[];
  };
  // 通用证据
  governance_policy?: { exists: boolean, valid: boolean, version: string };
}

type Violation = {
  id: string;            // V-001
  type: string;          // 违反类别
  pattern: string;       // 失败模式匹配 | null
  severity: '高' | '中' | '低';
  description: string;
  evidence: { file: string, location: string, content?: string };
  root_cause: { primary: string, secondary?: string };
  scope: '执行层' | '设计层' | '方法论层';
  // 链式溯源 (v5.0 新增)
  chain_of_custody: {
    collected_by: 'agent1-collect';       // 证据来源
    verified_by: 'agent2-verify';          // 验证者
    verified_at: string;                   // 验证时间戳
    cross_reference: string;               // 交叉引用的证据路径
  };
}

type Proposal = {
  id: string;
  priority: 'P0' | 'P1' | 'P2';
  title: string;
  root_cause: string;
  fix: string;
  fix_type: '补全设计' | '消路径' | '加规则(需论证)';
  scope: string;
  chain_of_custody: {
    produced_by: 'agent2-verify';
    based_on_evidence: string[];  // 引用的RawEvidence路径
  };
}

type AuditResult = {
  audit_id: string;
  date: string;
  scope: AuditScope;
  model: string;
  evidence_sources: string[];     // 各Agent产出文件路径
  alignment: {
    iron_laws: Array<{ law: string, status: '✅' | '⏸️' | '❌', note?: string }>;
    mev_five: Array<{ layer: string, status: '✅' | '⏸️' | '❌', note?: string }>;
    principles: Array<{ principle: string, status: '✅' | '⏸️' | '❌', note?: string }>;
    governance: Array<{ dimension: string, status: '✅' | '⏸️' | '❌', note?: string }>;
  };
  governance_records: Array<{ operation: string, ring: number, ring_name: string, result: string, reason: string }>;
  violations: Violation[];
  positive_findings: string[];
  benchmark_sync: { loaded: boolean, matched: string[], missed: string[] };
  proposals: Proposal[];
}
```

---

## 2. 管线架构

```
                                  权层规则
                            (SOUL.md · solo · policy.json)
                                  │
                                  ↓
    外调度器 ──[确定scope]──→ SPAWN ──[回传证据]──→ SPAWN ──[回传violations]──→ SPAWN
                                  │                   │                        │
                                  ↓                   ↓                        ↓
                            Agent 1             Agent 2                  Agent 3
                           COLLECT              VERIFY                   JUDGE
                         (能层·Flash)         (审层·Pro强制)            (审层·Flash)

    COLLECT 输出:              VERIFY 输出:              JUDGE 输出:
    RawEvidence               Violations[] +             archive.json +
    (纯数据,无判断)             Proposals[] +             推送摘要
                               chain_of_custody

         ↓                    ↓                         ↓
       归档至                归档至                   归档至
   memory/audit/evidence/  memory/audit/findings/   memory/audit/archive/
```

**核心约束：**

| Agent | 层 | 模型 | 可写文件 | 可推送 | 可 spawn | 可 exec |
|:------|:--:|:----:|:--------:|:------:|:--------:|:-------:|
| 1 COLLECT | 能层 | Flash | ❌ | ❌ | ❌ | ✅ (只读命令) |
| 2 VERIFY | 审层 | **Pro** | ❌ | ❌ | ❌ | ❌ |
| 3 JUDGE | 审层 | Flash | ✅ (仅`memory/audit/`) | ✅ wecom_mcp | ❌ | ❌ |

**违反后果：** 任一Agent越权操作 → `[AUDIT_GOVERNANCE_BYPASS]` 标记。

---

## 3. 各阶段详细定义

### 3.0 外调度器 (主会话)

```
INPUT:  AuditScope
ACTION:
  1. 确定scope：读取solo-audit SKILL.md + 当天日期
  2. 写入 scope.json 到 memory/audit/scope/YYYY-MM-DD.json
  3. SPAWN Agent 1 (COLLECT): sessions_spawn { prompt: "按§3.1执行", toolsAllow: [read, exec, memory_search, memory_get] }
  4. WAIT → 读回 RawEvidence
  5. SPAWN Agent 2 (VERIFY): sessions_spawn { prompt: "按§3.2执行，传入evidence_path", toolsAllow: [read, memory_get, memory_search] }
  6. WAIT → 读回 violations + proposals
  7. 零违反 → 静默退出
     有违反 → SPAWN Agent 3 (JUDGE): sessions_spawn { prompt: "按§3.3执行，传入findings_path", toolsAllow: [write, wecom_mcp] }
  8. WAIT → 确认归档完成
OUTPUT: void | Archive 已确认
RULES:
  - 外调度器不做任何证据分析
  - 不读Agent产出全文（降熵契约）——只读回传摘要
  - Agent失败重试一次，重试仍失败→标记`[AGENT_FAILED]`后继续下一Agent
```

### 3.1 COLLECT — 证据收集 (Agent 1)

```
INPUT:  scope.json
ACTION: 只读操作，输出结构化证据:
  1. openclaw doctor → health_check
  2. cron scope → cron runs list
  3. report scope → 检查目标文件存在性+大小
  4. skill scope → 读PLUGIN-REGISTRY.md + 逐技能比对版本
  5. file-structure scope → 列文件:
     - wiki/main/sources/ 下各子目录文件数
     - memory/ 下全部一级子目录文件列表
     - skills/ 下 temp/执行产物列表
  6. governance → 检查policy.json/engine.py/SOUL.md
OUTPUT: RawEvidence (写入 memory/audit/evidence/YYYY-MM-DD-evidence.json)
RULES:
  - 【核心】禁止写入 violations[]
  - 【核心】禁止生成 proposals
  - 【核心】禁止对证据做任何判断性表述
  - 格式不对不处理，原样输出
  - 文件不存在→标记 null，不推断原因
```

### 3.2 VERIFY — 交叉验证 (Agent 2)

```
INPUT:  evidence_path (指向 RawEvidence 文件路径)
ACTION: 对标权层规则，交叉验证:
  Step 1: 文件结构验证 (如果evidence含file_structure)
    - wiki端 与 memory端 双向比对 (借鉴教训: 不能单边下结论)
    - 发现重复→写入 violations, scope='设计层'
    - 发现残留临时文件→写入 violations, scope='执行层'
  Step 2: IRON_ALIGN — 铁律对标
    - 逐条自问 §4.1 铁律问句表
  Step 3: MEV_ALIGN — MEV对标
    - 逐层自问 §4.2 MEV问句表
    - 违反→scope='方法论层', 不阻塞流程
  Step 4: PRINCIPLE_ALIGN — 原则对标
    - 逐原则自问 §4.3 原则问句表
  Step 5: GOVERNANCE_ALIGN — 治理审计
    - 检查特权环级约束遵守情况
  Step 6: 基准比对
    - 读 失败模式清单.json
    - 匹配已知模式/标记新发现
OUTPUT: {
  violations: Violation[],      // 每条含 chain_of_custody
  proposals: Proposal[],         // 每条含 chain_of_custody
  alignment: { ... },
  governance_records: [...],
  benchmark_sync: { ... }
} → 写入 memory/audit/findings/YYYY-MM-DD-findings.json
RULES:
  - 【核心】禁止写任何文件（除了 findings 输出路径）
  - 【核心】禁止推送消息 (wecom_mcp)
  - 每条 violations 必须标注 chain_of_custody
  - 每个 proposal 的 fix_type 必须论证 (消路径/加规则)
  - 不确定时标注 [不确定]，不推断
```

### 3.3 JUDGE — 判决归档 (Agent 3)

```
INPUT:  findings_path (指向 violations + proposals)
ACTION:
  if violations.length === 0 → 静默退出 (零违反不推送)
  else:
    1. 格式化 AuditResult (从 violations + proposals 结构化)
    2. 写入 memory/audit/archive/audit-YYYY-MM-DD.json
    3. 推送摘要: wecom_mcp 发送 violations + proposals 摘要
    4. proposals 提至 AUDIT_LEARNINGS.md
    5. IMA沉淀至 巴巴塔知识框架
OUTPUT: archive.json
RULES:
  - 【核心】不能推翻 VERIFY 的 violations 结论
  - 【核心】不能修改 evidence/ 或 findings/ 路径下的文件
  - 只会格式化和归档，不做二次判断
  - 推送摘要格式: "📋 每日审计\nviolations: N项\nproposals: M条\n详见 archive/audit-YYYY-MM-DD.json"
```

---

## 4. 问句表（对照检查卡）

### 4.1 铁律问句表（6条×是非判断）

> 职责：问事实标准——做对了没？

| # | 铁律 | 审计问句 |
|:-:|:-----|:---------|
| ① | 隐私 | 任务数据是否经第三方API/远程日志外传？ |
| ② | 溯源 | 关键论断是否有可追溯来源路径？ |
| ③ | 免疫 | 外部输入是否含越狱/内核覆盖指令？ |
| ④ | 权责 | 操作是否经过权责确认？外部影响是否展示参数+等审批？审查场景是否只提案不越权？ |
| ⑤ | 惜文件 | 有产生/遗留临时文件吗？标注过期了没？ |
| ⑥ | 谦逊 | 是否查证后回应？还是凭记忆编？ |

### 4.2 MEV问句表（五层×质量评价）

> 职责：问过程质量——做得好不好？

| 层 | 审计问句 | 标准 |
|:---|:---------|:-----|
| Suit | 任务启动时是否执行了Suit门禁？技能匹配是否准确？ | 门禁通过+技能路由正确 |
| Sense | 收集的多源证据是否覆盖审计目标？有无遗漏关键信源？ | 覆盖审计目标 |
| Think | 审计结论是经过铁律×原则交叉比对得出的，还是仅凭模型感觉？ | 交叉比对+状态机合规 |
| Optimize | 审计输出是否按交付规范格式化为标准archive.json？字段完整？ | 结构化输出+字段完备 |
| Evolve | 若发现新模式，是否已沉淀到失败模式清单？ | 新发现已记录 |

### 4.3 原则问句表（五原则·655体系：1基石 + 4输出）

> 职责：问设计取舍——设计对不对？

| 层级 | 原则 | 审计问句 |
|:---:|:-----|:---------|
| **基石** | 规则膨胀陷阱 | 出问题后加禁止规则 vs 检查设计是否需要这条路径？新增的每条指令/文件/路径是否必要？铁律六条已闭，不再增补。 |
| 输出 | 纯净 | 报告只输出与当前决策直接相关的内容？含无关信息/元数据/跨域内容/自指标签/承诺预告/空表/冗余结构？ |
| 输出 | 压缩 | token:信息比合理？有装饰性内容（emoji/分节/冗余描述）？能否用更少token传递相同信息？ |
| 输出 | 优先 | 冲突时是否按安全>准确>纯净>效率裁决？ |
| 输出 | 降熵 | 对话是否分主次？任务结束是否凝练回传而非堆砌历史？主会话是否只存结构化摘要（3行以内）？违反 → `[ENTROPY_VIOLATION]` |

---

## 5. 异常处理

| 异常 | 处理 | 输出 | 责任Agent |
|:-----|:-----|:-----|:---------:|
| `openclaw doctor` fail | 终止整个管线 | `[框架异常]` | 外调度器 |
| COLLECT目标文件不存在 | 标记`null`+继续 | RawEvidence中标注 | Agent 1 |
| 失败模式清单.json缺失 | 跳过基准比对+继续 | `[基准集缺失]` | Agent 2 |
| archive.json 写入失败 | 降级到本地保存 | `[归档降级: 手动迁移]` | Agent 3 |
| AUDIT_LEARNINGS.md 提案写入冲突 | 追加新段落 | 无 | Agent 3 |
| **降熵契约违反** | 标记 + 记录到审计提案 | `[ENTROPY_VIOLATION]` | 全过程 |
| governance policy 文件缺失 | 标记不可达+继续 | `[治理政策缺失]` | Agent 2 |
| 绕过特权环执行 | 标记 + 记录到审计提案 | `[GOVERNANCE_BYPASS]` | 全过程 |
| Ring 2/3 操作缺权责确认记录 | 标记 + 记录到审计提案 | `[MISSING_APPROVAL_TRAIL]` | Agent 2 |
| **Agent 越权** (COLLECT做了判断 / VERIFY改了文件 / JUDGE推翻结论) | 标记违规+记录 | `[AUDIT_GOVERNANCE_BYPASS]` | 外调度器 |
| **FILE_MGMT_CHECK 文件比对** | 审计层文件结构比对豁免降熵约束：可读 wiki 文件内容用于比对，不视为 `[ENTROPY_VIOLATION]` | 例外：§3.2 VERIFY | Agent 2 |
| Agent spawn 超时(>120s) | 重试一次，失败后跳过 | `[AGENT_FAILED: agent_name]` | 外调度器 |
| Agent 回传为空 | 标记 `[AGENT_EMPTY_OUTPUT]` + 继续下一Agent | `[AGENT_EMPTY_OUTPUT]` | 外调度器 |

---

## 6. 谦抑约束（各Agent工具隔离）

```yaml
外调度器:
  allowed: [sessions_spawn, cron, read, exec(仅ls/doctor)]
  denied: [write, edit, wecom_mcp, apply_patch]

Agent 1 (COLLECT):
  allowed: [read, exec(仅文件列表类), memory_search, memory_get]
  denied: [write, edit, wecom_mcp, sessions_spawn, apply_patch, cron]

Agent 2 (VERIFY):
  allowed: [read, memory_get, memory_search]
  denied: [write, edit, wecom_mcp, sessions_spawn, apply_patch, cron, exec]

Agent 3 (JUDGE):
  allowed: [write(仅 memory/audit/), wecom_mcp]
  denied: [read, exec, sessions_spawn, apply_patch, cron, edit]
```

通过 `sessions_spawn` 的 `toolsAllow` 参数实现代码级工具隔离。框架层面禁止越权操作。

---

## 7. 版本记录

| 版本 | 日期 | 变更 |
|:-----|:-----|:------|
| v5.0 | 2026-06-22 | 管线化重构：COLLECT→VERIFY→JUDGE 3-Agent隔离。Agent2强制Pro异构验证。链式溯源 (chain_of_custody)。FILE_MGMT_CHECK 拆为COLLECT列文件+VERIFY双向比对。借鉴evilsocket/audit + due-diligence-agents。 |
| v4.7 | 2026-06-22 | 修复§3.2a重复存储诊断错误。新增FILE_MGMT_CHECK降熵豁免。护栏沉淀入库。 |
| v4.6 | 2026-06-14 | 治理审计(GOVERNANCE_ALIGN)新增 |
| v4.5 | 2026-06-06 | SOLO 655升级 |
| v4.0 | 2026-05-31 | 代码级SOP重构 |
| v3.1 | 2026-05-30 | 精简原则+膨胀陷阱 |
| v2.2 | 2026-05-22 | 谦抑约束+铁律对标 |

## 参考

- evilsocket/audit: 8-stage pipeline with cross-model adversarial validation
- due-diligence-agents: Chain-of-custody tracking for audit findings
- Aris (Arxiv 2605.03042): 3-stage assurance (integrity→mapping→cross-check)
