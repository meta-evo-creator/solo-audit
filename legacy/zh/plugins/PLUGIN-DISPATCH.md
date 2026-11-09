# Plugin Dispatch System  
> SOLO v1.0 — 内核精确定址机制 + 法纪自动升级  
> 位置：Suit 层 Step 0.5（在预检之后、五层执行之前）

## 内核级规则（不可跳过）

**法纪自动升级**：任务含任一法纪关键词 → 任务等级至少 L2。此规则在 Dispatch 之前执行。

法纪关键词表：`{纪检, 监察, 纪律, 法务, 法规, 处分, 问责, 巡视, 巡察, 党纪, 政纪, 八项规定, 四风, 三重一大}`

---

## 工作原理

任务进入 Suit 层时，遍历所有非 dormant 插件的 `trigger_fingerprint`，按关键词 + 反关键词 + 任务级别 + 优先级四维匹配，输出激活清单。

## Dispatch 流程

```
任务输入
  │
  ├─ Step 0: Kernel Boot (OpenClaw 原生 session startup)
  │
  ├─ Step 0.5: Plugin Dispatch
  │     │
  │     ├─ 1. 扫描所有 active/ + scene/ 插件的 trigger_fingerprint
  │     ├─ 2. 对每个插件进行四维匹配：
  │     │     [keywords ∩ 任务词汇] ≠ ∅  AND
  │     │     [anti_keywords ∩ 任务词汇] = ∅  AND
  │     │     [任务tier ∈ task_types]          AND
  │     │     [无更高priority的同功能插件冲突]
  │     ├─ 3. 候选插件按 priority 排序
  │     ├─ 4. 输出激活清单
  │     └─ 5. 显式标注：
  │         🔌 Dispatch: med-research(p5) + stage-checkpoint(p8) + deep-research(p3)
  │
  └─ Step 1-5: 五层正常执行
```

## 匹配规则

| 规则 | 说明 |
|:-----|:-----|
| **命中优先** | 只要 keywords 中任一词命中任务核心描述，即为候选 |
| **反排除** | anti_keywords 中任一词出现→立即排除该插件 |
| **级别限定** | 插件只在其声明的 task_types 范围内激活 |
| **优先级裁决** | 两个同类插件都匹配时，priority 高的胜出 |
| **强制激活** | stage-checkpoint 在 L2/L3/Cron 中 priority=10，不可被覆盖 |

## Chain 机制（v7.1 新增）

插件输出 JSON 中含 `trigger_next` 字段 → 内核自动 Dispatch 下一个插件。

```
插件A 输出: { result, quality, trigger_next: "政法分析" }
  → 内核 Gate 检查 quality 达标 → Dispatch 政法分析
```

典型链: deep-research → 政法评估（政策评估）
编排器: policy-eval 插件（L3/priority 4）

## 指纹格式

每个 `.plugin.md` 文件头部新增 `trigger_fingerprint` 块：

```yaml
# trigger_fingerprint:
#   keywords: []
#   anti_keywords: []
#   patterns: []
#   task_types: []
#   priority: 1-10
#   force_activate: true/false
```

| 字段 | 说明 |
|:-----|:-----|
| keywords | 触发该插件的关键词/短语 |
| anti_keywords | 排除词——任务含这些词时插件不激活 |
| patterns | 场景模式——比关键词更精准的模式匹配 |
| task_types | 允许的 MEV 级别：[L1, L2, L3] |
| priority | 优先级 1-10（10=系统级强制） |
| force_activate | 是否无视所有匹配规则、必须激活 |

## 与人脑判断的关系

Dispatch 不是取代人的判断——是**缩小候选集**。最终激活清单是建议，人可手动增删。格式：

```
🔌 Dispatch: med-research(p5) + stage-checkpoint(p8)
📋 候选但未激活: deep-research(p3) — 与 med-research 功能重叠
❌ 排除: compliance-research(无关键词命中) | bias-check(priority低于med-research)
✋ 用户可否: 回复 "+deep-research" 或 "-med-research" 手动调整
```
