# PLUGIN-REGISTRY.md — 技能索引表

> SOLO Fleet Registry — OpenClaw Skills 索引表。
> 技能统一存放于 `skills/` 目录，由 OpenClaw 框架通过 SKILL.md 的 `description` 关键词自动发现激活。
> 三轴分类体系参见 [`FLEET-CLUSTER.md`](../FLEET-CLUSTER.md)
> 最后更新：2026-06-16（加入三轴簇分类）

---

## Framework Plugins（OpenClaw 框架级）

| 插件 | 用途 |
|:-----|:------|
| memory-core | 记忆管理（短时+长时 + dreaming） |
| memory-wiki | 编译知识库WIKI |
| deepseek | 模型适配 DeepSeek V4 |
| wecom-openclaw-plugin | 企业微信通道 |
| openclaw-qqbot | QQ 机器人通道 |
| camofox-browser | 反检测浏览器引擎 |

## Scene（场景激发）

| 场景 | priority | keywords | task_types |
|:-----|:--------:|:---------|:----------|
| deep-research | 3 | 深度调研,技术评估,行业分析,多源交叉 | L3 |

## OpenClaw Skills（技能自动发现）

| 技能 | emoji | 簇 | 型态 | 环 | 触发词 | 架构 |
|:-----|:-----:|:--:|:----:|:-:|:------|:-----|
| solo | ⛳ | ⚙️工具 | 元技能 | 2 | Fleet巡检,系统架构,三权制衡,内核审计 | 三权制衡 v2.1(内核+审计+技能) |
| solo-audit | 🔍 | 🔒审计 | 元技能 | 3 | 每日审计,文件结构审计,铁律对标,根因分析 | solo-audit v5.0.0(3Agent管线·链式溯源·异构验证) ⛔保密专属 |
| discipline-inspection | ⚔️ | 🏛️大监督 | 管线编排 | 3 | 纪律审查,案件定性,处分建议,谈话提纲,信访核查,执纪审理,违规认定,归责分析 | 8-Agent管线(Scope→Search→Audit→Analyze→Draft→Review→Revise→Publish) v1.0.0 ⛔保密专属 |
| supervision-inspection | 🔍 | 🏛️大监督 | 管线编排 | 3 | 巡视巡察,政治体检,巡视报告,问题底稿,四个落实,巡视整改,一把手监督 | 8-Agent管线(Scope→Search→Audit→Analyze→Draft→Review→Revise→Publish) v1.1.0 ⛔保密专属 |
| hospital-inspection | 🏥 | 🏛️大监督 | 管线编排 | 3 | 医院巡查,大型医院巡查,巡查报告,问题整改,党建巡查,行风巡查,运行管理巡查,巡查反馈,巡查底稿,医院自查 | 8-Agent管线(Scope→Search→Audit→Analyze→Draft→Review→Revise→Publish) v1.0.0 ⛔保密专属 |
| compliance-analysis | 🏛️ | 🏛️大监督 | 管线编排 | 3 | 合规分析,采购合规,招投标合规,科研合规,数据合规,医保合规,个人信息保护,合规评估,合规咨询,风险定级 | 8-Agent管线(Scope→Search→Audit→Analyze→Draft→Review→Revise→Publish) v1.0.0 ⛔保密专属 |
| babata-browser | 🦞 | 🦞采集 | 工具封装 | 1 | 学术搜索,政府网站,JS渲染,截图,web_fetch失败 | CloakBrowser v3.2.0 |
| babata-superocr | 📝 | 🦞采集 | 工具封装 | 1 | 图片OCR,聊天截图,截图文字,PDF扫描件,手写体识别 | PaddleOCR+RapidOCR v2.0.0 (PDF+截图+批量) |
| solo-file-transfer | 📤 | ⚙️工具 | 工具封装 | 2 | Word转Markdown,上传IMA,文件上传知识库 | docx_to_md + ima_upload.cjs v1.1.0 |
| party-review | ⚖️ | 🔬研究 | 管线编排 | 2 | 党建评审,党建好案例,党建评选,四维评分,一句话点评 | 七阶段评审(Validate→Parse→Score→Recommend→Draft→Review→Deliver) v1.1 ⛔保密专属 |
| msf | ⚒️ | 🔬研究 | 管线编排 | 2 | 医学研究,系统评价,证据质量,团标,技术路线图,学会 | OPL stage-led v4.2.3 |
| deep-research | 🔍 | 🔬研究 | 管线编排 | 2 | 深度调研,技术评估,行业分析,多源交叉,L3任务 | 8-Agent管线(Scout→Merge→Analyze→Draft→Review→Revise→Publish) v4.0.0 |
| babata-search | 🌐 | 🦞采集 | 工具封装 | 0 | 中文搜索,百度,360,搜狗,国内信息检索 | Camoufox+Playwright双引擎 v2.0 |
| officecli-xlsx | 📊 | ⚙️工具 | 工具封装 | 1 | .xlsx表格,Excel数据处理,报表,数据分析 | Rust单二进制+OpenXML v1.0.116 |

## 未启用/待安装
| 插件名 | 说明 | 状态 |
|:-------|:-----|:----:|
| blogwatcher | 博客监测（RSS/Web） | 已安装但未完成配置 |
