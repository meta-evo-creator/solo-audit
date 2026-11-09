# Edge TTS 配置（免费·中文女声）
> 2026-07-24 · TTS工具集名发现

## 启用方法
cron enabled_toolsets: `tts`（不是 text_to_speech）
```yaml
enabled_toolsets: ["web","terminal","file","delegation","tts"]
```
## 配置
`hermes config set tts.provider edge`
`hermes config set tts.edge.voice zh-CN-XiaoxiaoNeural`
## 陷阱
- `text_to_speech` 作为工具集名 → 不可用
- `tts` → 可用
- 中文女声: zh-CN-XiaoxiaoNeural · 5000字符上限