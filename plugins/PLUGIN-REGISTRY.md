# PLUGIN-REGISTRY.md

> SOLO Fleet Registry — Plugin注册表 + OpenClaw Skills索引。
> 新能力 → scene/ → 触发≥3次 → active/ → 30天无触发 → dormant/ → 删除

---

## Active（常加载）

| 插件 | priority | 触发条件 | 最后触发 |
|:-----|:--------:|:---------|:--------:|
| tool-awakening | 10(f) | 每次Suit层（含技能识别门禁+文件预检+搜索降级） | 2026-05-21 |
| core-lessons | 10(f) | 每次Suit层 | 2026-05-21 |
| cron-rules | 10(f) | 隔离cron会话 | 2026-05-14 |

## Scene（按需激活）

| 插件 | priority | keywords | task_types |
|:-----|:--------:|:---------|:----------:|
| deep-research | 3 | 深度调研,技术评估,行业分析,多源交叉 | L3 |

## OpenClaw Skills（框架级自动激活）

| 技能 | emoji | 触发词 | 架构 |
|:-----|:-----:|:------|:-----|
| discipline-inspect | ⚔️ | 巡视,纪检,监察,纪律,党纪,问责,整改,监察建议,九项准则 | 4-Agent独立子代理(sessions_spawn) + ripgrep(17法规.md) |
| MSF | ⚒️ | 医学研究,系统评价,Meta分析,临床试验,证据,论文 | Scout→Draft→Review(审稿分离) |
| babata-browser | 🦞 | 政府网站,JS渲染,截图,web_fetch失败 | CloakBrowser v3.1 |
| babata-superocr | 📝 | 手写体识别,PDF扫描件,OCR | PaddleOCR+RapidOCR |
| solo-file-transfer | 📤 | Word转Markdown,上传IMA,文件上传知识库 | docx_to_md + ima_api 二合一 |

---

## Dispatch 规则

1. 任务进入 Suit → 遍历 active + scene 插件 + OpenClaw skills
2. OpenClaw skills 由 `<description>` 关键词自动激活（框架级强制）
3. 内部插件通过 keywords 匹配 + `read` 加载
4. force_activate=true → 跳过匹配
5. conflict: discipline-inspect(p6) ∩ MSF(p5) → discipline-inspect 优先
