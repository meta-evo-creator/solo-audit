# Edge TTS Configuration (free · Chinese female voice)
> 2026-07-24 · discovery of the TTS toolset name

## How to enable
cron enabled_toolsets: `tts` (not text_to_speech)
```yaml
enabled_toolsets: ["web","terminal","file","delegation","tts"]
```
## Configuration
`hermes config set tts.provider edge`
`hermes config set tts.edge.voice zh-CN-XiaoxiaoNeural`
## Pitfalls
- `text_to_speech` used as the toolset name → not available
- `tts` → available
- Chinese female voice: zh-CN-XiaoxiaoNeural · 5000-character limit
