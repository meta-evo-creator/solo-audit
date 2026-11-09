# Cron 修改护栏

## 铁律
修 cron 基础设施（路径/时间/密钥）时，**不得同时修改内容格式**。

## 案例
2026-07-21 修复三个报告 cron 的 Hermes 路径时，同时修改了报告模板——从原版情报内参格式变成含 MEV 矩阵、DELIVERY CHECK、偏倚自检的治理文档。

## 正确做法
- 修路径：改 absolute path、search tool、workdir
- 修时间：改 schedule
- 修内容：单独提 PR，不混入基础设施修复

## 验证
改完 cron → 立即 `cronjob run` → 检查产出格式是否与前一天一致
