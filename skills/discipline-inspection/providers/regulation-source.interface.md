# Regulation Source Interface v1.0

> **知识源可插拔接口规范** — DI技能管线通过此接口与法规数据层解耦。
> 任何实现本接口的 Provider 都可以驱动 DI 管线。

---

## 接口定义

每个 Provider 通过 `provider.yaml` 声明自身能力和配置：

```yaml
name: "provider-name"
version: "1.0"
description: "简要描述"

capabilities:
  regulation_search: true|false      # 是否支持法规全文检索
  version_verification: true|false   # 是否支持法规版本/时效性验证
  case_search: true|false            # 是否支持指导性案例检索
  methodology_access: true|false     # 是否支持方法论文档访问

degradation:
  missing_version_verification: "VERSION_UNVERIFIED | BLOCK | SKIP"
  missing_regulation_search: "BLOCK | USE_DEFAULT"
  note: "当某能力缺失时的降级行为说明"

search:
  engine: "ripgrep | pkulaw-mcp | custom"
  base_path: "${VARIABLE}"           # 环境变量，agent 解析时替换
  script: "path/to/script.py"        # (可选) CLI 脚本路径
  scopes:                            # 搜索域映射
    discipline_laws: "${BASE}/法规"
    medical_laws: "${BASE}/medical"
    cases: "${BASE}/指导性案例"
    methodology: "${BASE}/方法论"
```

---

## 能力矩阵

| 能力 | Agent 消费者 | 缺失时管线行为 |
|:-----|:------------|:--------------|
| `regulation_search` | Agent 1a (search-rg) | 若 `USE_DEFAULT` → 回退到 default-provider；若 `BLOCK` → 管线拒绝启动 |
| `version_verification` | Agent 1b (search-pkulaw) | 所有法规标记 `VERSION_UNVERIFIED`，Agent 2/3 附加 ⚠️ 警告 |
| `case_search` | Agent 1a | 该项跳过，不阻断管线 |
| `methodology_access` | Agent 1a / Agent 3 | 降级：从 SKILL.md 内嵌方法论引用 |

---

## Provider 选择逻辑

管线启动时，主会话（或 Agent 0）按以下优先级选择 Provider：

```
1. 环境变量 DI_REGULATION_PROVIDER 指定 → 使用指定 provider
2. 检测 WIKI_PATH 环境变量存在 + wiki 目录可读 → 使用 wiki-provider
3. 检测 pkulaw-mcp 可用 → 叠加启用 pkulaw-provider（版本验证增强）
4. 以上均不可用 → 使用 default-provider（内置示例法规包）
```

Provider 之间可以**叠加**：
- `wiki-provider` 提供法规检索（Agent 1a）
- `pkulaw-provider` 提供版本验证（Agent 1b）
- 两者独立配置，互不依赖

---

## 内置 Provider 清单

| Provider | 能力 | 适用场景 |
|:---------|:-----|:---------|
| **default-provider** | 法规检索（3部核心法规）+ 方法论 | 开源用户开箱即用 / demo |
| **wiki-provider** | 法规全文检索 + 案例检索（45+部法规） | 有本地WIKI法规库的机构 |
| **pkulaw-provider** | 法规版本验证（现行有效/已修改/废止） | 有北大法宝订阅的机构 |

---

## 自定义 Provider

用户可以创建自己的 Provider：
1. 在 `providers/` 下新建目录
2. 编写 `provider.yaml`（按上述接口）
3. 设置环境变量 `DI_REGULATION_PROVIDER=your-provider-name`
4. 管线启动时自动加载

---

## 环境变量

| 变量 | 说明 | 默认值 |
|:-----|:-----|:-------|
| `DI_REGULATION_PROVIDER` | 指定 provider 名称 | auto-detect |
| `WIKI_PATH` | WIKI法规库根路径 | (无) |
| `SKILL_DIR` | DI技能目录 | 自动检测 |
