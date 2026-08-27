# OpenClaw→Hermes 迁移损失恢复日志

> 2026-07-20 | 巴巴塔
> 每次平台迁移必然伴随知识损失。本文档记录从 OpenClaw 恢复的所有关键资产和教训。

## 恢复的文件

| 文件 | 路径 | 内容 |
|:-----|:-----|:-----|
| HEARTBEAT.md | hermes/ | 舰队监控·文件预算·每日审计触发 |
| MEMORY.md | hermes/ | WIKI唯一性护栏·审计同步规则·大监督共享 |
| TOOLS.md | hermes/ | Tavily双Key轮换·搜索降级链·反爬规则 |
| PLUGIN-REGISTRY.md | hermes/ | 技能最后触发时间 |
| FLEET-CLUSTER.md | hermes/ | 技能三轴分类 |
| IDENTITY.md | hermes/ | 身份核心 |
| 约束效率平衡.md | memory/goals/ | 长期目标追踪 |
| 2026-07-04-findings.json | memory/audit/findings/ | 审计发现(膨胀陷阱) |
| SP-001-rejected.md | memory/audit/proposals/ | 驳回提案记录 |
| SOLO体系更新v5.1 | memory/system/ | 关键方法论升级 |
| forum-collaboration-sop.md | memory/system/ | Forum协作SOP |
| workspace-file-management.md | memory/system/ | WIKI写入五守卫 |

## 恢复的 cron

| cron | 描述 |
|:-----|:-----|
| 法规周检（每周一08:00） | 5部核心法规版本检查，与月检互补 |
| 每日临时文件清理（03:00） | /tmp/、桌面残留、delegation缓存 |

## 硬化的护栏

| 护栏 | 文件 | 变化 |
|:-----|:-----|:-----|
| 防幻觉硬约束 | search-rg.md | 条款号必须从 rg 提取，禁凭记忆 |
| 主体要件门禁 | search-rg.md | Step 1.5 恢复 |
| Citation <80% REJECT | agent4-review.md (MSF) | 代码级 grep |
| 谦逊检查 | agent4-review.md (MSF+DR) | 自动 REJECT |
| Human Gate <65 | MSF+DR+DI SKILL.md | OPL 人因门禁 |

## 核心教训

1. **内存记忆 ≠ 文件事实** — 永远读源文件，不凭训练记忆
2. **"知识库里有的，先看再用"** — 巡视底稿教训适用所有技能
3. **rg 未命中 ≠ 法规不存在** — 必须走完整降级链
4. **出问题先检查执行，不加规则** — AUDIT-20260704 核心发现
5. **cron 改配置必须同步 prompt 版本号** — 2026-06-27 教训
