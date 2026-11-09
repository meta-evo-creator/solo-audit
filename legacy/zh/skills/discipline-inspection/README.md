# Discipline-Inspection ⚔️ 纪律审查

> **纪律为准绳，持续守望。** 8-Agent 管线 × 违规+有责双因素分析法 × 二十四字方针6维评分矩阵。

纪检监察案件分析引擎：接受案件事实 → 法规检索 → 版本验证 → 审计 → 深度分析 → 报告撰写 → 质量评审 → 终版产出。

---

## 🚀 快速开始

### 开箱即用（demo模式）

```bash
git clone <repo-url>
cd discipline-inspection
# 无需任何配置，默认知识包包含3部核心法规
```

将案件事实输入 Agent 0（Scope），管线自动运行。产出包含 ⚠️ 版本未验证标记。

### 生产使用

```bash
# 1. 配置法规库
export WIKI_PATH=/path/to/wiki/main/sources

# 2. (可选) 配置版本验证
# 安装 pkulaw-search 技能 + pkulaw-mcp 服务
```

配置后管线自动获得完整45+部法规检索 + 北大法宝版本验证能力。

---

## 三层价值

| 层 | 不需要知识源 | 需要默认知识包 | 需要完整WIKI+PKULaw |
|:--:|:------------|:-------------|:-------------------|
| 🥇 **方法论** | ✅ 违规+有责双因素 / 二十四字方针评分矩阵 | — | — |
| 🥈 **管线架构** | ✅ 8-Agent File Handoff / Guardrail Routing | — | — |
| 🥉 **可运行实例** | — | ✅ 3部核心法规demo | ✅ 45+法规+案例+版本验证 |

---

## 管线架构

```
Phase 0: Scope       → 问题界定
Phase 1a: Search-rg    → 法规全文检索 (Provider驱动)
Phase 1b: Search-pkulaw → 版本验证 (可选，无PKULaw降级)
Phase 2: Audit       → 法规引用审计
Phase 3: Analyze     → 违规+有责双因素分析
Phase 4: Draft       → 报告/提纲撰写
Phase 5: Review      → 二十四字方针6维评分
Phase 6: Revise      → 修复
Phase 7: Publish     → 终版产出
```

**三模式分流：**
- `full`: 案件定性/处分建议 (8 Agent)
- `interview`: 谈话提纲 (5 Agent)
- `quick`: 法规咨询/条款查询 (4 Agent)

---

## 知识源架构

DI 管线通过 **Provider 接口** 与法规数据层解耦：

```
┌─────────────────────────────────────────┐
│           DI 8-Agent 管线               │
│  (方法论 + 分析框架 + 评分矩阵)          │
└──────────┬─────────────┬────────────────┘
           │             │
    ┌──────▼──────┐ ┌───▼──────────┐
    │ 法规检索     │ │ 版本验证      │
    │ Provider    │ │ Provider     │
    └──────┬──────┘ └───┬──────────┘
           │             │
    ┌──────▼──────┐ ┌───▼──────────┐
    │ default     │ │ pkulaw       │
    │ wiki        │ │ (北大法宝)    │
    │ custom...   │ │              │
    └─────────────┘ └──────────────┘
```

详见：[`providers/regulation-source.interface.md`](providers/regulation-source.interface.md)

---

## 依赖说明

### 运行时依赖
- **OpenClaw** — Agent 管线运行时
- **ripgrep** (`rg`) — 法规全文搜索
- **Python 3** — pkulaw-search 脚本（可选）

### 可选依赖
| 依赖 | 能力 | 缺失时行为 |
|:-----|:-----|:----------|
| WIKI法规库 (45+部) | 完整法规检索、指导性案例 | 降级为3部核心法规demo |
| pkulaw-mcp | 法规版本/时效性验证 | 所有法规标记 VERSION_UNVERIFIED，不阻断管线 |
| 北大法宝订阅 | pkulaw-mcp 数据源 | 同上 |

---

## 目录结构

```
discipline-inspection/
├── SKILL.md                    # 技能完整规格
├── README.md                   # 本文件
├── agents/                     # 8个Agent prompt
│   ├── scope.md
│   ├── search-rg.md
│   ├── search-pkulaw.md
│   ├── audit.md
│   ├── analyze.md
│   ├── draft.md
│   ├── review.md
│   └── revise.md
├── providers/                  # 知识源可插拔层 🔌
│   ├── regulation-source.interface.md
│   ├── default/                # 内置示例法规包
│   │   ├── provider.yaml
│   │   └── knowledge/
│   ├── wiki/                   # WIKI法规库适配器
│   │   └── provider.yaml
│   └── pkulaw/                 # 北大法宝适配器
│       └── provider.yaml
└── references/                 # 参考文档
    └── scoring-matrix.md
```

---

## 方法论

### 违规+有责双因素分析框架

```
违规（客观行为要素）  ×  有责（主观归责要素）  =  综合量纪
├─ 行为事实              ├─ 主观状态（故意/过失）
├─ 法律依据              ├─ 认知程度
├─ 客体侵害              ├─ 动机目的
├─ 持续性                ├─ 事后态度
└─ 情节程度              └─ 身份叠加
```

### 二十四字方针6维评分

定性准确(25%) · 事实清楚(20%) · 证据确凿(20%) · 处理恰当(15%) · 手续完备(10%) · 程序合规(10%)

---

## 许可证

本仓库包含：
- 管线代码/方法论/评分框架：MIT License
- 法规文本（`providers/default/knowledge/`）：来源于中国政府公开法律文件，属于公共领域信息
- 完整法律文本以全国人大/国务院官方发布版本为准

---

## 免责声明

本工具输出的分析结论为**候选参考**，不可替代：
- 纪委会议/审批程序的正式决定
- 纪检监察干部的专业判断
- 法律专业人士的意见

使用者应自行核实法规版本时效性和事实准确性。
