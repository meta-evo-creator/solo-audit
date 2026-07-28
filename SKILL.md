---
name: "solo-audit"
description: "SOLO Audit v6.6 — TAFA-Native + Kanban审批: 七铁律对标+四引擎心跳+学习闭环。四Agent管线。⛔ Confidential."
platforms:
  - hermes
tools:
  - read
  - exec
  - cron
  - delegate_task

metadata:
  hermes:
    emoji: 🔍
    pipeline: delegate_task
    delivery: native_qqbot
---

# SOLO 审计 Agent v6.5

> TAFA完整闭环：权层定规则 → 能层执行 → 审层审计·发现违规 → ⏵路由能层 → ①能层提案 → 权层(Human in the Loop)审批 → ①能层修复 → ②能层护栏(归因·定位·固化·验证) → ③审层下次验证
> v6.5：审层严格收窄——只发现违规+路由，kanban卡标题"⏳待分析"不含方案。提案是能层的活。
>
> | 等级 | 流程 | 判定 |
> |:----:|:-----|:-----|
> | 🔴P0 | 路由权层·人工审批 | 隐私/权责/数据丢失/生产影响 |
> | ⚠️P1 | 路由能层·自动提案·审批后执行 | 路径/格式/验证/阈值 |
> | 🔵OBS | 路由能层·仅记录 | 性能波动/趋势/非紧迫 |

---

## 0. 协议签名

```yaml
PROTOCOL: solo-audit v6.0
入口:    { trigger: 'cron' | 'direct', scope: TAFAScope }
出口:    AuditResult | void(零违反)

TAFA 三层对齐:
  审层职能: 审核执行层（Agent/cron）是否遵守立法层（石冰·655六铁律）规则
  审计范围: 铁律对标 + 基础设施 + 系统状态 + 学习闭环
```

## 审计范围（审层 · 755(七铁律+五原则·膨+MEV五层)）

### 一、铁律审计
扫描cron日志搜索违规模式。②溯源(无来源>20%⚠️) | ④权责(跳过Ring2🚨) | ⑦诚实(降级无声明🚨) | ③⑥常规检查。跨日复现→P0升级。

### 二、原则审计
膨胀陷阱(新规则≠消路径) | 纯净(核心文件月增长≤15%) | 压缩(同信息不重复3+处) | 优先(P0提案72h) | 降熵(主会话≤80%+Memory≤90%)。

### 三、MEV审计
Suit(纪检任务主会话直接执行🚨) | Sense(跳过WIKI⚠️) | Think(跳过增强链⚠️) | Optimize(未评分或<60🚨) | Evolve(教训未转化⚠️)。

### 四、Agent性能评估（新增·observation only）
监控能层各Agent的运行健康度。性能差≠违规——只记录，不判决。
- 幻觉率：无来源声明占总输出的比例（>15%⚠️）
- 延迟：各Agent平均响应时间（>基准2倍⚠️）
- 重试率：管线恢复重试次数（>3次⚠️）
- 数据写入 `collected_data.json` 的 `agent_performance` 字段
三引擎心跳(Tavily/pkulaw/IMA) | cron状态 + 技能目录 + CNKI Cookie | 上期提案执行状态(>7天P0)。

---

## 外调度器（主会话 → cron触发时自动执行）

```
模型: deepseek-v4-flash
工具: [delegate_task, cron, read, terminal, file]
职责: 确定scope → 顺序delegate 4个Agent → 聚合产出 → 交付

交付协议:
  1. 零违反 → 静默退出（无推送）
  2. 有违反 → 最终回复即审计摘要（cron推送到QQ）
  3. P0提案 → 特别标注⚡
```

## TAFA 闭环流程

```
审层(审计)                    能层(修复)                    权层(审批)
   │                             │                             │
   ├─ 发现违规                    │                             │
   ├─ 路由至能层[x] ──────────→   │                             │
   │                             ├─ 分析根因                    │
   │                             ├─ kanban create 提案 ────→    │
   │                             │                             ├─ 看Kanban待审批
   │                             │                             ├─ 石冰说"批准"
   │                             │              ✅通过/❌打回 ← ┘
   │                             ├─ 执行修复 ←  ─ ─ ─ ─ ─ ┘
   │                             ├─ 护栏式沉淀                  │
   │           ← ─ 反馈完成 ─ ─ ┘                              │
   └─ 下次审计验证                                              │
```

核心纪律：
- 审层：只审计·只路由·不改代码·不提方案
- 能层：只提案·只执行·不审批。修复后必须护栏式沉淀——把教训固化为永久规则
- 权层：只审批·不定向·不审计·不执行
- 护栏式沉淀三步：①归因（是什么类型的错误）→ ②定位（哪个位置可以拦住）→ ③固化（写入cron/skill/AGENTS.md）→ ④触发验证（针对修复点做一次复测，确认护栏生效）

## Agent 1 (COLLECT — 能层): 数据收集 · 使用 execute_code 批量

**工具**: `execute_code` 批量读取+分析（替代逐个terminal调用）

**输入**: scope → **输出**: `collected_data.json`
2. **能层心跳（真探测）**: 每个远程服务做一次端到端真实查询（非curl HEAD/import）：
   - Tavily: `python scripts/tavily_search.py "test" --limit 1` → 验证返回非空结果
   - pkulaw: `python scripts/pkulaw_search.py law --title "宪法" --json` → 验证返回非空
   - IMA: `ima_api.cjs search_knowledge_base "test"` → 验证响应200+非空
   - CNKI: `python scripts/cnki_search.py "测试" --limit 1` → 验证cookie有效，非CAPTCHA页
   - 结果写入 `{service: {status, latency_ms, result_count, error}}`
3. **执行数据**: cronjob list / 技能目录存在性 / 文件中版本号
4. **闭环数据**: 读上期审计提案文件 → 提取未完成项+滞留天数

```
输出:

{
  "权层日志扫描": {
    "②溯源表述": [{file, line, text, level}],
    "④权责跳过": [{file, line, text, level}],
    "⑥未验证声明": [{file, line, text, level}]
  },
  "跨日复现": {提案ID: 复现状态},
  "能层心跳": {
    "tavily": {status, latency_ms, result_count, error},
    "pkulaw": {status, latency_ms, result_count, error},
    "ima": {status, latency_ms, error},
    "cnki": {status, latency_ms, cookie_valid, error}
  },
  "执行合规": {crons, skills},
  "闭环数据": {提案ID: 滞留天数}
}
```

> 真探测 vs 假心跳：`curl HEAD` 只能证明端口开着；`python search "test"` 能证明工具真正可用。本地工具（babata-search/browser/superocr）不做远程心跳——它们出问题会在执行时暴露，且更新不快。

## Agent 2 (VERIFY — 权层): 验证 · 禁止建议

**输入**: collected_data.json → **输出**: `verification_report.json`

- 逐条验证 COLLECT 数据是否准确
- 排除假阳性（日志中的"记不清"如果是用户原文引用=非Agent违规）
- 跨日复现确认：上期违规项本期是否再次发生

## Agent 3 (ANALYZE — 审层): 分类 · 路由 · 建卡

**工具**: `kanban` 创建"⏳待分析"卡

**输入**: verification_report.json → **输出**: 分类路由结果

- 违规分类（个人/系统性/基础设施）
- 按三级分流标注 🔴P0 / ⚠️P1 / 🔵OBS
- **路由**：每条标注能层接收方
- **Kanban建卡**：`hermes kanban create "⏳待分析: [违规简述]" "[等级] [事实] · 路由至: 能层[xxx]"`
- **❌ 不给修复方案**——方案、根因、操作步骤是能层的活
- **❌ 不写"📋 提案"**——那越界了
- 审层只发现+路由+建卡。修改由权层审批后能层执行

## Agent 4 (REPORT — 交付): TAFA 三层标注输出

### 输出格式（强制）

每条违规必须用 TAFA 三层关系标注：**能层[动作] → 违反 → 权层[规则]**。

```
🚨 能层 → 违反 → 权层铁律②（溯源为本）
  能层动作：cron报告第3段输出无来源标注
  权层规则：每句关键论断须有可追溯来源
  证据：read_file(报告, L45-48)
  等级：⚠️

🚨 能层 → 违反 → 权层纯净护栏
  能层动作：输出包含"15:00 CST"——编造的时间戳
  权层规则：输出禁止编造时间戳/工具状态/版本号
  证据：read_file(报告, L12)
  等级：🔴
```

### 标注规则 + 闭环执行
- 格式：`🚨 能层[动作] → 违反 → 权层[规则]`
- 每行必须有：能层动作 + 权层规则原文 + 工具证据
- 每条违规必须带 `⏵ 路由至：能层[xxx] · 建议修复：[xxx]`
- 无证据 → 标注[未证实] → 不得列为违规
- 零异常 → [SILENT]

### 护栏式沉淀提案（每条违规必须附）
修复完成后，能层必须将教训固化为护栏。审计输出最后附沉淀清单：
```
🛡️ 护栏沉淀提案：
- 归因：[错误类型]
- 定位：[在哪个位置加护栏]
- 固化：[写入哪个文件/规则]
- 状态：⏳ 待能层执行
```

### 🔴 输出前自检
逐条问：能层做了什么？违反了权层哪一条？我亲眼看到了吗？
三条答不上任一 → 删除该条。

### 📋 审计末尾必须输出
```bash
hermes kanban list
```
将 Kanban 待审批清单附在报告最后：
```
【KANBAN 待审批】
  t_xxxxx: ⏳ [标题]
  (无待审批项 =「全部已审批」)
```
