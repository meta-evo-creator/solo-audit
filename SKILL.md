---
name: "solo-audit"
description: "SOLO Audit v6.9 — TAFA-Native + Kanban审批: 七铁律对标+现行五原则(7/25精炼·8/25定版)+MEV对标+结构性护栏实测。四Agent管线。⛔ Confidential."

metadata:
  hermes:
    emoji: 🔍
    pipeline: delegate_task
    delivery: native_qqbot
---

# SOLO 审计 Agent v6.9

> TAFA完整闭环：权层(SOLO755宪法) → 能层执行 → 审层审计·发现违规 → ⏵路由能层 → ①能层提案 → 权层(Human in the Loop)审批 → ①能层修复 → ②能层护栏(归因·定位·固化·验证) → ③审层下次验证
> v6.9：**五原则审计对齐权层现行版（2026-08-26）**——旧版"纯净/压缩"过时（7/25 精炼为"精简"·8/25 认知定版加入"护栏"）——原则审计段修正为：膨胀陷阱/精简/优先/降熵/护栏（护栏实测见第五节）。
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
PROTOCOL: solo-audit v6.9
入口:    { trigger: 'cron' | 'direct', scope: TAFAScope }
出口:    AuditResult | void(零违反)

TAFA 三层对齐:
  权层: SOLO 755宪法 = 七铁律 + 五原则(膨胀/精简/优先/降熵/护栏) + MEV五层(Suit/Sense/Think/Optimize/Evolve)。由石冰(Human in the Loop)持有。
  能层: 14技能 + 子代理管线 + cron任务。执行提案·修复·护栏沉淀。
  审层: solo-audit。审核能层是否遵守权层(SOLO755)规则。只发现+路由，不决策。
  审计范围: 铁律对标 + 原则对标 + MEV对标 + 基础设施 + 学习闭环
```

## 审计范围（审层 · 对标权层 SOLO755宪法）

### 一、铁律审计
扫描cron日志搜索违规模式。②溯源(无来源>20%⚠️) | ④权责(跳过Ring2🚨) | ⑦诚实(降级无声明🚨) | ③⑥常规检查。跨日复现→P0升级。

**🔴 ④权责分明·用户文件越权专项（2026-08-07·TAFA落地）**：
审层智能体必须检查能层是否越权操作用户文件：
- 扫描 `logs/audit_hooks.log` 中的 `[BLOCKED]` 记录——有=能层试图越权·被结构性拦截 → 审计记录+评估能层是否反复尝试
- 扫描 cron/脚本输出中是否出现 `/mnt/c/`·用户业务目录的写/删/移操作（未被拦截的漏网）→ 🚨 P0
- 检查能层是否在未获石冰明确指令时触碰用户文件（`/mnt/c/`·Desktop/Documents/Downloads/OneDrive·聊天记录·企业微信数据）→ 🚨 P0
- **判定标准**：权层SOLO755规定"AI只动自己管理的文件·用户文件结构性禁止"——任何违反即P0，不受"用户口头允许"影响（口头"清理/迁移/删除"≠授权）

### 二、原则审计（对照权层现行五原则·2026-07-25 精炼·2026-08-25 认知定版）
**膨胀陷阱**(出问题先查已有机制是否执行·新规则≠消路径·加规则前三问) | **精简**(制度少而精·冗余技能/机制/文档) | **优先**(抓主要矛盾·P0提案72h·高风险先管低风险不折腾) | **降熵**(主会话≤80%+Memory≤90%·信息流清晰·留痕) | **护栏**(结构性不靠自觉——见第五节实测验证·告示牌≠护栏)。

### 三、MEV审计
Suit(纪检任务主会话直接执行🚨) | Sense(跳过WIKI⚠️) | Think(跳过增强链⚠️) | Optimize(未评分或<60🚨) | Evolve(教训未转化⚠️)。

### 四、Agent性能评估（新增·observation only）
监控能层各Agent的运行健康度。性能差≠违规——只记录，不判决。
- 幻觉率：无来源声明占总输出的比例（>15%⚠️）
- 延迟：各Agent平均响应时间（>基准2倍⚠️）
- 重试率：管线恢复重试次数（>3次⚠️）
- 数据写入 `collected_data.json` 的 `agent_performance` 字段
引擎+双手心跳(Tavily/pkulaw/IMA/CNKI/可可做/可可问) | cron状态 + 技能目录 | 上期提案执行状态(>7天P0)。

### 五、🔴 结构性护栏验证（v6.8新增·TAFA落地检验）
**TAFA落地的检验标准=护栏在·而非自觉在**。审层必须验证结构性护栏真实生效（不是看文档写没写·是实测拦不拦）：

| 护栏 | 验证方法（实测） | 失效判定 |
|:-----|:----------------|:---------|
| **Ring 99 用户文件保护** | `python3 scripts/policy-engine/engine.py delete_file --path /mnt/c/Users/shibi/Desktop/x.md` → 应返回 Ring 99 user-file-protected·allowed=false | 返回allowed=true或非Ring99 → 🚨P0 护栏失效 |
| **audit_hook预工具拦截** | 向audit_hook.py喂rm用户文件命令 → 应退出码1+[BLOCKED] | 退出码0 → 🚨P0 护栏失效 |
| **Ring 3 技能操作拦截**（2026-08-26） | 喂 `skill_manage create`（无授权）→ 应退出码1+Ring 3 提示 | 退出码0 → 🚨P0 护栏失效 |
| **Ring 3 cron 操作拦截**（2026-08-26） | 喂 `cronjob remove`（无授权）→ 应退出码1+Ring 3 提示 | 退出码0 → 🚨P0 护栏失效 |
| **Ring 3 config 操作拦截**（2026-08-26） | 喂 `hermes config set`（无授权）→ 应退出码1+Ring 3 提示 | 退出码0 → 🚨P0 护栏失效 |
| **audit_hooks.log有[BLOCKED]记录** | 扫描最近100行含[BLOCKED] → 说明拦截在真实工作 | 0条且近期有越权尝试 → ⚠️P1 记录异常 |

**验证结果写入 `collected_data.json` 的 `guardrails` 字段**（每个护栏: {status: 生效/失效, test: 实测命令, result}）。
**护栏失效=权层规则失去架构保障=最高级P0**——不依赖能层自觉补位，立即路由权层。引擎+双手心跳(Tavily/pkulaw/IMA/CNKI/可可做/可可问) | cron状态 + 技能目录 | 上期提案执行状态(>7天P0)。

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

## Agent 1 (COLLECT — 审层): 数据收集 · 使用 execute_code 批量

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

### 2026-08-26: kanban CLI 全拦="会话标记污染"·先查 env 再判断
来源：石冰审批 3P1+2OBS 后建卡·`hermes kanban create/show/complete` 全部报 "delegate_task child contexts cannot mutate Kanban tasks via the CLI"·同会话 8/24 还能 complete·8/26 突然全拦。
**根因**：kanban_db.py `_assert_not_delegated_child_mutation()` 靠环境变量 `HERMES_DELEGATED_CHILD_CONTEXT=1` 识别子代理上下文（delegate_task 子进程唯一标记）。主会话派完子代理后·Hermes 把会话沿袭标记为 child lineage·**terminal 持久环境被注入该标记**——之后所有 terminal 命令都"看起来像子代理"→ kanban CLI 拒绝。gateway 进程环境/ bashrc 均干净（验证排除法确认污染在会话 terminal 环境）。
**修复**：`unset HERMES_DELEGATED_CHILD_CONTEXT && hermes kanban ...` 立即恢复（主会话有权利·这是修正误判非绕过护栏——护栏对象是子代理·真子代理环境无法 unset 到父会话）。
**判定流程**：kanban 全拦 → ①`env | grep DELEGATED`（命中=标记污染）②确认当前是真主会话（QQ 直聊/cron 主调度）③unset 后重试 ④若标记来自 gateway 本身（/proc/<pid>/environ 含标记）→ gateway 是被子代理上下文启动的·需重启 gateway 根治。
**验证**：unset 后 kanban list/create/complete 全恢复·3 卡补建归档成功。

### 2026-08-12: 路径扫描只扫 SKILL.md = 漏掉子代理执行文档（doc_path_scan 首跑即发现）
来源：doc_path_scan 首跑报 8 处旧Windows路径·修完 SKILL.md 后 grep 全库 → agents/modes/pipeline/*.md 还有 11 个文件 30+ 处 `C:\Users\shibi\AppData\Local\hermes\...`（子代理实际照 agents/*.md 执行·比 SKILL.md 更常撞）
**根因**：①doc_path_scan 只 rglob("SKILL.md")·agents/modes/pipeline 漏扫 ②子代理执行文档比 SKILL.md 更关键（搜索命令全在 agents/*.md）
**修复**：①doc_path_scan 扩展扫描全部 .md（SKILL.md+agents+modes+pipeline）+ scripts/*.py|*.sh·按文件去重 ②替换规则：`C:\Users\shibi\AppData\Local\hermes\workspace\skills\X\scripts` → `~/.hermes/skills/X/scripts`（workspace 层不存在）、`...\wiki\main\sources\` → `~/.hermes/wiki/main/sources/` ③verify-merge-coi.py 默认路径旧Windows → `os.path.expanduser("~/.hermes/tmp/...")` ④排除词扩展（git-bash/MSYS/cmd.exe/Startup/junction/PowerShell/New-Item/管理员/实测/引号等）防经验说明文字误报·双向±2行 context 判断
**教训**：
- 扫描范围必须覆盖"子代理实际会执行的文件"（agents/modes/pipeline/scripts）·非仅主 SKILL.md
- 修复后必须反向验证：临时注入旧路径确认扫描器能捕获（防过度过滤导致真问题漏网）
- 路径映射注意 workspace\skills\X 实际是 skills/X（迁移后无 workspace 层）·不可机械替换
- 回归验证：4 脚本全绿 + 临时文件注入测试通过

### 2026-08-12: 审计查"存在"≠查"功能"——工具健康需实测（自愈缺口根治）
来源：石冰追问"采集技能可用吗"→ 实测发现 pkulaw 语法失效+sherlock 缺包——DO 管线 Scout 曾踩到（日志: pkulaw-mcp search → unknown command）但静默降级 web_search·无人发现
**根因**：①Scout 工具失败静默降级·无反馈 ②审计只查脚本存在性·不查正确性（pkulaw 存在但语法错·sherlock 存在但缺包）
**解决**：`~/.hermes/scripts/tool_health_check.py`（实际调用 9 工具·验证"能工作"非"存在"）→ 接入 skill_health_check 4.7 → 每日审计自动跑
**教训**：
- 自愈 = 审计能抓到"工具坏了"（功能级）·非仅"文件没了"（存在级）
- 工具故障必须反馈回路（_lessons.json 收集·审计报警·自主修复）
- 修复模式：实测发现 → 定位语法/依赖 → 修技能文档 → 验证全绿 → 沉淀

### 2026-08-12: 技能进化管理·自反式防膨胀（三分法+监控+清理）
来源：58技能+52条LEARNED堆积（deep-research 12/msf 9/writing-convention 8）→ 石冰问"如何管理进化避免膨胀"
**三分法（写LEARNED前自问）**：
| 类型 | 处理 | 示例 |
|:-----|:-----|:-----|
| 版本历史(vX.X更新) | ❌ 不写LEARNED·进changelog | "v4.2改了什么" |
| 一次性修复(代码已改) | ❌ 修复即完·不沉淀 | PATH bug修完 |
| 可复用教训(会再遇) | ✅ 写·带日期+评审点 | 石冰纠正类 |
**监控**：`python3 ~/.hermes/scripts/skill_health_check.py`（LEARNED>8/重叠/久未更新/超大备份→报警·已接入审计cron第四步）
**清理**：2026-08-12 52→33条（-36%·删版本流水账·留真教训）·备份在~/.hermes/backups/skills-leaned-20260812/
**教训**：减法也是进化·护栏在而非自觉在（curator备份56.7MB膨胀就是无人监控的盲区·健康检查抓出）
**审计建议**：每日审计可加一项"脚本测试状态"（pytest 全绿 = 护栏在）

### 2026-08-12: 脚本测试化落地（OPL/last30days 研究·真实能力提升）
来源：deep-study last30days-skill（57.9k⭐·190测试）→ 我们脚本零测试是最大短板 → 落地 pytest
现状：`~/.hermes/scripts/.testvenv/bin/python -m pytest tests/` = **32 passed**（3个文件）
覆盖：validate_pipeline_json（10用例·schema校验）/ audit_hook（17用例·安全护栏）/ heartbeat（5用例·平台适配）
**真实价值**：audit_hook 测试当场暴露 2 个安全漏洞（Remove-Item Windows路径漏拦 + echo重定向写用户区漏拦）→ 已修复（2026-08-12 patch）
**教训**：
- 测试不是形式——它抓住了"只读判断没考虑重定向写入"和"Windows路径正则缺失"两个真漏拦截
- 给关键脚本加测试 = 护栏式改进（防回归·不膨胀）
- 运行方式：`.testvenv/bin/python -m pytest tests/ -q`（uv 建的 venv·pytest 9.1.1）
**审计建议**：每日审计可加一项"脚本测试状态"（pytest 全绿 = 护栏在）


### 2026-08-10: cron no_agent 脚本 PATH 同源缺陷（第3个中招）
来源：能力雷达 cron 手动触发仍失败（heartbeat.py "不可用: Tavily, pkulaw, IMA"）——主会话跑正常·cron 跑失败。
根因：cron no_agent 环境 PATH = `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`，**缺 `~/.local/bin`**（pkulaw-mcp 在此）→ subprocess.run 找命令 FileNotFoundError。
已中招脚本链：①weekly_backup.sh (8/9) ②cleanup_temp.py (8/10) ③heartbeat.py (8/10) ④memory-consolidation.py 旧Windows路径 (8/10)。
**教训：所有被 cron no_agent 调用的脚本（.py/.sh）必须自查开头是否有 PATH 补全**：
```python
os.environ['PATH'] = '/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:' + os.path.expanduser('~/.local/bin') + ':' + os.environ.get('PATH', '')
```
**验证方法**：`env -i HOME=/home/shibi python3 script.py`（空环境模拟 cron）——不是普通 python3 跑。
**审计盲区**：audit_collect.py 的 heartbeats 检查用主会话 PATH 测，测不出 cron 环境问题——必须用空环境模拟。审计改进：heartbeats 检查加空环境验证。

### 2026-08-10: 审层机制已被覆盖时·主会话加规则=膨胀陷阱
来源：石冰纠正"每日审计本来就对照SOLO755，里面就包含Evolve"——主会话想给审计cron加"教训小结"步骤，实际 solo-audit 早已含 MEV审计（Evolve教训未转化⚠️）+ 学习闭环（修复后护栏式沉淀）。
教训：
- **加规则前先 grep 已有技能/脚本是否覆盖**——solo-audit v6.8 的"MEV审计"章节已检查 Evolve（教训未转化⚠️），审计范围已含"学习闭环"
- 主会话对审层机制的了解浮于表面——skill_view 返回压缩摘要时，必须 read_file 读全文再下结论
- 正确动作：审计 prompt 无需新增步骤，只需确保既有 Evolve 机制被**实际执行**（而非文档存在但未跑）
- 沉淀为通用规则：**「加机制前三问」①已有技能是否覆盖？②是否违反五原则？③最小实现？全过才加**
