# Agent Reach 架构对标分析

> 来源：调研 Panniantong/Agent-Reach（59k★, 4.7k forks, v1.5.0）
> 日期：2026-07-23
> 用途：为 SOLO 舰队采集簇和审计体系提供外部参考

---

## Agent Reach 核心理念

**能力层（Capability Layer），不是又一个工具。** 不读内容——负责选型、安装、体检、路由。内容读取由 Agent 直接调上游工具完成，零包装开销。

---

## 三个关键设计模式

### 1. Channel + 多后端有序路由

```
channels/twitter.py → twitter-cli ▸ OpenCLI ▸ bird
channels/bilibili.py → bili-cli ▸ OpenCLI ▸ 搜索API（yt-dlp 已被封，退役）
```

每个渠道 = 首选 + 备选的有序列表。换接入方式 = 调顺序，不用重写代码。后端坏了自动降级——B站封了 yt-dlp，切 bili-cli，用户零操作。

**对 SOLO 的启示**：采集簇每个搜索工具目前只有单一路径。可定义降级链：
- 中文搜索：babata-search → web_search native
- 国际搜索：tavily-search → web_search native
- 法规搜索：pkulaw-search → web_search + 北大法宝官网直接爬

### 2. 真探测（端到端测试）

`agent-reach doctor` 对每个后端做端到端测试——不是 `which twitter-cli`，而是真正发请求、验证返回数据。坏掉的给修复处方。

**对 SOLO 的启示**：舰队健康检查当前只查目录存在 + API HEAD 可达。应升级为：每个采集技能做一次真实查询，验证返回数据有效。已记录在 `fleet-health-check.md`。

### 3. 安装即注册（一键消除多表同步）

装完后自动在 Agent 的 skills 目录写 SKILL.md，Agent 以后遇到对应需求自己知道调哪个工具。

**对 SOLO 的启示**：我们目前 SOUL.md → FLEET-CLUSTER.md → PLUGIN-REGISTRY.md 三表同步是 07-23 审计发现的漂移根源（缺口 5 个技能）。考虑将注册信息收敛到 PLUGIN-REGISTRY.md 单表（已是 OpenClaw 原生格式）。

---

## 心跳范围原则（本次审计确立）

只检查远程服务（API 可能挂），不检查本地库（装了就不会突然消失）。

| 该查 | 不该查 | 原因 |
|:-----|:------|:-----|
| Tavily API | easyocr import | 远程可能挂；本地包安装后永不会失败 |
| IMA API | paddleocr import | 同上 |
| pkulaw API | yt-dlp 版本 | 同上 |
| CNKI cookie 过期 | Python 包版本 | cookie 会过期；包版本变化不阻塞 |

---

## 对标总表

| 维度 | Agent Reach | SOLO 舰队 | 差距 |
|:-----|:-----------|:----------|:-----|
| 多后端路由 | 每渠道首选+备选列表 | 每工具单一路径 | 无降级链 |
| 健康检查 | 端到端真实探测 | 目录存在+API HEAD | 不知工具是否真正可用 |
| 技能注册 | 安装→SKILL.md→自动发现 | 手工维护三张注册表 | 07-23 发现 5 个遗漏 |
| 故障恢复 | 后端切换用户无感 | 需要人发现→手动修 | 被动 |
| 零配置开箱 | 6 个渠道零配置 | 每个技能独立配置 | 新技能接入门槛高 |
