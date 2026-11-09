# IMA上传路径幻觉 · 已固化的护栏

> 发现：2026-07-23 审计 SP-030 | 修复：同日闭环

## 问题

能层cron在执行IMA上传时，声称 `upload_to_kb.cjs` 不存在（实际文件存在）。
根因：MSYS路径格式(`/c/Users/...`)与Node.js期望的Windows路径(`C:/Users/...`)不匹配。
Agent未验证即报告"脚本不存在"——违反了诚实为根。

## 护栏（已固化至三个cron payload）

```
上传前验证步驟：
1. ls "C:/Users/shibi/AppData/Local/hermes/skills/ima-skill/upload_to_kb.cjs" — 确认文件存在
2. cd "C:/Users/shibi/AppData/Local/hermes/skills/ima-skill"
3. node upload_to_kb.cjs "KB_ID" "" "C:/Users/shibi/AppData/Local/hermes/memory/..." "title" 2>&1
4. 验证输出含 "success" 或 "ok" 或 media_id → 报告✅
5. 失败 → 报告真实错误原因，不编造
```

## 通用原则

已部署确认的脚本路径 → 直接执行 + 失败报错。
不要用 search_files 验证 → 路径格式不一致会产生假阴性。
Node.js 需 Windows 绝对路径 (`C:/...` 非 `/c/...`)。
