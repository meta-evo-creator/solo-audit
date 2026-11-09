---
name: "solo-audit"
version: 6.13.0
description: "SOLO Audit v6.13 — TAFA-Native + Kanban审批: 七铁律对标+现行五原则(7/25精炼·8/25定版)+MEV对标+结构性护栏实测+留痕核查(OBS·脚本化)。四Agent管线。⛔ Confidential."

metadata:
  hermes:
    emoji: 🔍
    pipeline: delegate_task
    delivery: native_qqbot
---

# SOLO 审计 Agent v6.13

> **v6.14 增补（2026-11-04·石冰批）**：新增第七节「运行时环境一致性」（第 24 项）——`scripts/env_consistency.py` → `audit_collect.py::check_env_consistency`，回归门 `scripts/tests/test_env_consistency.py`。由来：TTS 连续两天 403 而 23 项全绿，因静态判据看 pyproject pin 与开发 venv，Hermes 实跑在 pm 的 `installs/…/venv`（「判据与现实脱节」第 11 例）。判据只认真运行时版本 + 真调一次；optional 未装不判违规；判据缺失一律 ⚠️ 不得 ✅。
> **v6.13 增补（2026-10-03·石冰批 t_1180ba7c）**：第六节「留痕核查」由**模型人肉判断**改为**脚本产出**（`scripts/audit_collect.py::check_trace` → `scripts/trace_scan.py`，逻辑与回归测试同源）；扫描范围补 `workspace/**/io_stage`——原口径只指 `tmp/·reports`，致真实案件 `_trace.md` 在盘却连报"0 办案任务"= **假通过**（「判据与现实脱节」第 8 例）。已随本次修复新增回归用例 `scripts/tests/test_trace.py`。
> **v6.12 增补（2026-09-28）**：两条**回归门自检**规矩——① `scripts/tests/` 只放 pytest 用例，一次性校验/勘误脚本放 `scripts/checks/`；② 全量回归**必须 `pytest <目录>` 整目录跑**，禁止以单文件通过当回归证据；报告「通过」前先确认门真的被收集执行（`no tests ran` = 失败）。理由见 LEARNED「判据与现实脱节」家族实例 6。
> TAFA完整闭环：权层(SOLO755宪法) → 能层执行 → 审层审计·发现违规 → ⏵路由能层 → ①能层提案 → 权层(Human in the Loop)审批 → ①能层修复 → ②能层护栏(归因·定位·固化·验证) → ③审层下次验证
> v6.9：**五原则审计对齐权层现行版（2026-08-26）**——旧版"纯净/压缩"过时（7/25 精炼为"精简"·8/25 认知定版加入"护栏"）——原则审计段修正为：膨胀陷阱/精简/优先/降熵/护栏（护栏实测见第五节）。
> **v6.10 增补（2026-09-23）**：新增第六节「留痕核查（OBS级）」——对应 `supervision-shared`「能层留痕口径」（石冰拍板·能层自定）；审层只核查、不制定口径。
> **v6.9 增补（2026-09-13·石冰厘清审层职责边界）**：本审计 = **对照权层要求审计能层合规**。**不是所有具体问题的修复都通过本审计**——具体修复走能层正常闭环（提案→审批→执行→护栏沉淀），审计只负责「发现违规 + 路由」，不包办、不代办。
> **由此自守一条（反博弈）**：**审计判据/阈值属权层标准，不得为了让告警好看而改度量方式**。例：技能体量告警刺眼，就把判据从字节改字符让它消失——那是**改尺子不是改问题**。要动判据，唯权层发话；否则修能层实际状态。
> v6.8：**护栏有效性验证**——审层不只发现违规，还验证结构性护栏（Ring 99用户文件保护·audit_hook预工具拦截·policy-engine）是否真实生效。TAFA落地的检验标准=护栏在·而非自觉在。
> v6.7：三层归属修正——**权层=SOLO 755宪法（七铁律+五原则+MEV五层）**。审层全部Agent（COLLECT/VERIFY/REPORT）均为审层内部步骤；ANALYZE为能层。权层=石冰本人（Human in the Loop），不设Agent。
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
PROTOCOL: solo-audit v6.13
入口:    { trigger: 'cron' | 'direct', scope: TAFAScope }
出口:    AuditResult | void(零违反)

TAFA 三层对齐:
  权层: SOLO 755宪法 = 七铁律 + 五原则(膨胀/精简/优先/降熵/护栏) + MEV五层(Suit/Sense/Think/Optimize/Evolve)。由石冰(Human in the Loop)持有。
  能层: 14技能 + 子代理管线 + cron任务。执行提案·修复·护栏沉淀。
  审层: solo-audit。审核能层是否遵守权层(SOLO755)规则。只发现+路由，不决策。
  审计范围: 铁律对标 + 原则对标 + MEV对标 + 基础设施 + 学习闭环
```

## 审计范围（审层 · 对标权层 SOLO755宪法）

> 本节只写**判据**（什么算违规、什么等级）。检查项明细（实测命令/表格/写法）已逐字移入
> `references/audit-scope-v613.md`（2026-10-26 瘦身 · 卡 t_6349b2e6）——需要动手核时先查该文件。

### 一、铁律审计
扫描cron日志搜索违规模式。②溯源(无来源>20%⚠️) | ④权责(跳过Ring2🚨) | ⑦诚实(降级无声明🚨) | ③⑥常规检查。跨日复现→P0升级。
**④用户文件越权专项**：能层越权操作用户文件（`C:/`·Desktop/Documents/Downloads/OneDrive·聊天/企微数据）= 🚨P0，**不受"口头允许"影响**（口头"清理/迁移/删除"≠授权）；`logs/audit_hooks.log` 的 `[BLOCKED]` 记录须评估能层是否反复尝试。明细→refs §一。

### 二、原则审计（对照权层现行五原则·2026-07-25 精炼·2026-08-25 认知定版）
**膨胀陷阱**(出问题先查已有机制是否执行·新规则≠消路径·加规则前三问) | **精简**(制度少而精·冗余技能/机制/文档) | **优先**(抓主要矛盾·P0提案72h·高风险先管低风险不折腾) | **降熵**(主会话≤80%+Memory≤90%·信息流清晰·留痕) | **护栏**(结构性不靠自觉——见第五节实测验证·告示牌≠护栏)。

### 三、MEV审计
Suit(纪检任务主会话直接执行🚨) | Sense(跳过WIKI⚠️) | Think(跳过增强链⚠️) | Optimize(未评分或<60🚨) | Evolve(教训未转化⚠️)。

### 四、Agent性能评估（observation only）
性能差≠违规——只记录，不判决。判据：幻觉率>15%⚠️ | 延迟>基准2倍⚠️ | 重试>3次⚠️；结果写 `collected_data.json.agent_performance`。明细→refs §四。
引擎+双手心跳(Tavily/pkulaw/IMA/CNKI/可可做/可可问) | cron状态 + 技能目录 | 上期提案执行状态(>7天P0)。

### 五、🔴 结构性护栏验证（v6.8新增·TAFA落地检验）
判据：**TAFA 落地的检验标准=护栏在·而非自觉在**——不看文档写没写，看实测拦不拦。结构性护栏逐个实测（Ring 99 用户文件保护·audit_hook 预工具拦截·Ring 3 的 skill/cron/config 拦截·`audit_hooks.log` 有 `[BLOCKED]` 记录）；返回 `allowed=true` 或**退出码 0 = 🚨P0 护栏失效**，即权层规则失去架构保障，最高级 P0，立即路由权层。结果写 `collected_data.json.guardrails`（{status, test, result}）。实测命令与失效判据表→refs §五。

### 六、留痕核查（**P1 级·2026-10-11 石冰批·t_4bb43876**）
> 口径来源 `supervision-shared`「能层留痕口径」（能层自定）；审层**只核查、不制定口径**。沿革：v6.10 先记 OBS → 连续 4 轮零缺口触发升级判据#1 → 2026-10-11 权层批准**升 P1**（自下一轮起 `_trace.md` 缺失即判违规）。

判据：`_trace.md` 缺失 → ⚠️P1 | 同一阶段产物被重复写入/覆盖改写（留痕只追加不覆盖）→ ⚠️P1（证据链污染·原话零改写） | 含案件个人数据的产物外流（IMA/第三方API/外部知识库）→ 🚨P0。结果写 `collected_data.json.trace`。核查项表→refs §六。

### 升级判据（**自触发**·按审计轮次计·不按自然周）
- **台账**：`state/留痕核查台账.json`——每轮追加一行 `{date, 办案类任务数, trace缺失数, 覆盖改写数, L2外流数, 结论}`。
- **升 P1 条件（满足其一）**：①连续 4 轮审计零缺口（`SOLO每日审计` 为日度 → 4 轮即 4 天）；**已于 2026-10-11 触发并获批·本条不重复触发**，若后续降级再现缺口则按新台账重新累积。②**到期强制复核 2026-10-31**——当日无论台账状态，须专门复核「留痕口径是否真在执行」并出结论（生效/未生效/需改口径）。
- **提案≠改状态**：升级＝出提案走权层（石冰批），审层不自行升格；未获批期间继续 OBS 记录。
- **台账丢失或超 30 天未更新** → 视为未达标，同样触发第②条复核。明细→refs 升级判据。


---

## 外调度器（主会话 → cron触发时自动执行）


```
模型: 审计档 `kimi-k3`（provider `kimi-coding-cn`·由 cron `SOLO每日审计` 342d1edde6b9 钉住；技能不硬编码·由 host config / cron 决定）
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

## Agent 1 (COLLECT — 审层): 数据收集 · 使用 execute_code 批量


**工具**: `execute_code` 批量读取+分析（替代逐个terminal调用）

**输入**: scope → **输出**: `collected_data.json`
2. **能层心跳（真探测）**: 每个远程服务做一次端到端真实查询（非curl HEAD/import）：
   - Tavily: `python scripts/tavily_search.py "test" --limit 1` → 验证返回非空结果
   - pkulaw: `python scripts/pkulaw_search.py law --title "宪法" --json` → 验证返回非空
   - IMA: `ima_api.cjs search_knowledge_base "test"` → 验证响应200+非空
   - CNKI: `python scripts/cnki_search.py "测试" --limit 1` → 验证cookie有效，非CAPTCHA页
   - 结果写入 `{service: {status, latency_ms, result_count, error}}`
   - 🔴 **证据闸②（2026-10-22 权层批准·t_9d2a3c27）**：远程探针**单次失败不得直接判 P1**。
     脚本内由 `audit_collect._confirm()` 统一处理：首探失败 → 复跑 1 次；任一次通过 ⇒ `status=✅ + transient=True + level=OBS`（报告标 OBS 并注明首探失败）；连续 2 次皆败 ⇒ `status=❌ + confirmed_fail=True + level=P1` 才可升正式告警。
     依据：2026-10-15 曾把 sherlock/numpy 的瞬时失败判成 P1，同日同解释器复跑全绿 = 误报。
   - 🔴 **证据闸①③**：凡来自**历史台账/执行记录**的 P1 断言，先复核证据「当前态」（cron 失败须过 `_fixed_after_fix()`：该 job 自己引用的脚本 mtime 晚于失败时刻 ⇒ 标 RESOLVED，不报 P1）；已 RESOLVED 的项下一轮不得重复报 P1。验收用例：`scripts/tests/test_audit_gates.py`（①—⑧）。
   - 🔴 **计数口径**（2026-10-22 连带修）：采集簇可用性只能数「✅ 工具名: …」**明细行**，且 ❌ 按工具名**去重**；
     否则汇总行被各抓一次 → 9 个工具报成 10/10、1 个故障报成 2 个（曾因此把 8/9 报成 8/10）。
     pkulaw-mcp 判据已改 `pkulaw_ok()`（认「命中约」/pkulaw.com 链接，兼容「成功」），不复用字面串。
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

## Agent 2 (VERIFY — 审层): 验证 · 禁止建议


**输入**: collected_data.json → **输出**: `verification_report.json`

- 逐条验证 COLLECT 数据是否准确（审层内部步骤·非权层）
- 排除假阳性（日志中的"记不清"如果是用户原文引用=非Agent违规）
- 跨日复现确认：上期违规项本期是否再次发生
- 🔴 **子代理环境注意（2026-08-26 教训）**：delegate 子代理缺 `XDG_RUNTIME_DIR`——检查 systemd 用户级服务（dsh-qqbot 等）必须显式设置 `env={**os.environ, 'XDG_RUNTIME_DIR': f'/run/user/{os.getuid()}'}`·否则误报 "Unit could not be found"（8/26 VERIFY 曾把 dsh-qqbot 误报为"裸进程非托管"·主会话复核纠正）。凡 systemctl --user 检查一律带此 env。

## Agent 3 (ANALYZE — 能层): 分析 · 提案 · 建卡


**工具**: `kanban` 创建"⏳待审批"卡

**输入**: verification_report.json → **输出**: 分析提案+kanban

- 违规分类（个人/系统性/基础设施）
- 按三级分流标注 🔴P0 / ⚠️P1 / 🔵OBS
- **分析根因并提案**：每条标注具体修复步骤+预期效果
- **Kanban建卡**：`hermes kanban create "⏳待审批: [具体修复提案]" "[分析] · 路由至: 能层[xxx]"`
- **⛔ 禁止建"⏳待分析"卡**——那是不完整的。必须分析完、提案完，才建卡。
- 修改由权层审批后能层执行

## Agent 4 (REPORT — 审层): TAFA 三层标注输出


### 输出格式（强制）

每条违规必须用 TAFA 三层关系标注：**能层[动作] → 违反 → 权层[规则]**。

```
🚨 能层 → 违反 → 权层铁律②（溯源为本）
  能层动作：cron报告第3段输出无来源标注
  权层规则：每句关键论断须有可追溯来源
  证据：read_file(报告, L45-48)
  等级：⚠️

🚨 能层 → 违反 → 权层精简护栏
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
**🔴 无待审批卡时必须输出「全部已审批」，禁止保留 `t_xxxxx` 占位符（2026-08-09 教训）**——占位符会被误读为真实待审批卡（8/8 审计曾把模板示例当成"5张待审批卡"汇报）。判据：`hermes kanban list` 输出无 `⏳`/`ready` 状态的卡 → 直接写「全部已审批」，不列出任何示例卡。

---

## LEARNED PATTERNS
- 🔴 **2026-10-29：「判据与现实脱节」家族 实例 11 + 12（法宝配额误判 / 交互式任务静默不跑）**
  ① **配额特征串要覆盖 CLI 现用文案**：`tool_health_check` 的 `QUOTA_MARKERS` 只有 `90001`/`checking remaining points`/`认证失败`，而 CLI 现在直接输出「法宝 MCP **积分不足**，请前往 … 充值」→ 配额耗尽被当成「工具故障」，还给出错误的「核对工具名/参数」建议（把人往错方向带）。**规矩**：凡按错误文案分流自愈的判据，必须把 CLI 当前全部同类文案收齐；分错流比不检测更糟（把可自愈的额度问题报成代码缺陷）。② **交互式计划任务在无用户登录时静默不执行**：`schtasks /run` 仍返回「成功」，但**任务「上次运行时间」不变、进程也不出现**——只看返回码=假通过。判据三件套：`query user`（有无登录用户）+ 任务「上次运行时间」是否刷新 + 目标进程/CDP 端口是否真的起来。本实例链条：无人登录 → `HermesEdgeSnapshot` 不跑 → CDP 9222 无响应 → `kkz_pkulaw_reward.py` 领不到当日 10000 积分 → pkulaw 全库检索不可用（法规一级来源断供）。**凡依赖「快照 Edge / CDP 9222」的自愈链，都要先验「本机有没有登录会话」**。
- 🔴 **「判据与现实脱节」家族（4 实例合并·2026-09-13 → 09-18）**：凡**写死的标签表 / 选择器 / 解释器 / 临时文件名 / 目标路径**，都会随现实漂移 → 退化成两种假象：**假告警**（永远 ❌）或**假通过**（没测到却 ✅，更隐蔽）。**合并后规则**：
  ① **证据集取全**：断言认的标签必须列全源码中的同类标签（网关脏退出除 `exit_nonzero`/`exit_dirty`，还有 `gateway.previous_unclean_exit`）；无样本时退回核对「标签名是否仍在源码中」——源码中消失才算真盲区。
  ② **解释器同源**：验证/自测命令的解释器必须与 cron/网关同源（`hermes-agent/venv/Scripts/python.exe`）；判据要绑「能力所在的解释器」，不是「跑脚本的解释器」。
  ③ **选择器限定**：加 `no_agent` 假 + prompt 非空限定，避免命中 prompt 恒空的脚本任务。
  ④ **移走/还原一律 `os.replace()`**（Windows `Path.rename` 不覆盖已存在目标，会抛 `WinError 183`）；固定名临时文件须「启动清孤儿 + 幂等还原 + `atexit`/`finally` 兜底」三保险。
  ⑤ **移走失败必须打 stderr 并计为异常**，不得静默降级成「跳过该项」——假通过比假告警更糟。
  ⑥ 报告缺陷前先问一句：「这是产品的错，还是我验证方式的错？」
  ⑦ **修复后必须反向验证**：临时注入旧路径，确认扫描器能捕获（防过度过滤漏真问题）；扫描范围须覆盖「子代理实际会执行的文件」（agents / modes / pipeline / scripts），非仅主 SKILL.md；路径映射注意 `workspace\skills\X` 实为 `skills/X`（迁移后无 workspace 层），不可机械替换。
  **实例（备查）**：09-13 周深审（脏退出标签 + 心跳选择器）· 09-14 第二实例（heartbeat OCR 探测绑了调用者解释器 → 长期假告警）· 09-15 第 4 实例（自测用 Python312 报不存在的 `yaml` 缺失 → 自己的假阳性）· 09-18 第 5 实例（`rename` 移走失败 → `script_layer_*` 假全绿，`cron/executions.db` 10/10 completed 零 error，运行状态层完全不可见）。
### 2026-10-06: 技能每日更新检查污染探针 stdout（「判据与现实脱节」家族·实例 10）→ 逐字归档 `references/learned-patterns-archive.md`
> **规矩要点**（细则看归档）：① 探针判据不得依赖被测进程 stdout 的净度——先隔离（自持 `IMA_LAST_CHECK_FILE`）再判；② 探针不得改写被测对象自身状态；③ 回归门 `scripts/tests/test_ima_probe_env.py`。
> **兄弟型「采集 ≠ 判定」**：凡有阈值意义的指标，采集完必须判、异常落 `anomalies`；阈值要收敛告警面（结构性常驻项只进 detail，不得独立 ⚠️）；复用既有脚本不新增 cron；回归门 `scripts/tests/test_audit_usage_judge.py`。

> 📁 本节历史条目（27 条）已归档 → `references/learned-patterns-archive.md`


---

