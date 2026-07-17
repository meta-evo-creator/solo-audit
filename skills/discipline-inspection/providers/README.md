# Providers — 知识源可插拔

DI 技能管线通过 Provider 接口与法规数据层解耦。不同用户可以使用不同的知识源驱动分析管线。

## 快速选择

| 我想... | 用这个 Provider | 需要什么 |
|:--------|:---------------|:---------|
| 快速体验/demo | `default-provider` | 无需配置，开箱即用 |
| 机构生产使用 | `wiki-provider` | 本地WIKI法规库 |
| 法规版本验证 | `pkulaw-provider` | 北大法宝订阅 |

## 组合使用

Provider 可以叠加：
```
wiki-provider（法规检索）+ pkulaw-provider（版本验证）
```

也可以单独使用：
```
wiki-provider 单独 → 法规检索正常，版本标记 VERSION_UNVERIFIED
default-provider 单独 → 仅3部核心法规，版本标记 VERSION_UNVERIFIED
```

## 检测优先级

管线启动时自动检测，无需手动指定：
1. 检查环境变量 `DI_REGULATION_PROVIDER` → 使用指定 provider
2. 检查 `WIKI_PATH` → 存在则启用 wiki-provider
3. 检查 pkulaw-mcp → 可用则叠加启用 pkulaw-provider
4. 以上均不可用 → 使用 default-provider

## 自定义 Provider

创建你自己的知识源适配器：
1. 在 `providers/` 下新建目录
2. 编写 `provider.yaml`（按 `regulation-source.interface.md` 接口）
3. 设置 `DI_REGULATION_PROVIDER=your-name`

详见：`regulation-source.interface.md`
