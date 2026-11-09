# 655 进化指标基线

## evolution-metrics.json

路径: `memory/.mev/evolution-metrics.json`

追踪指标:
- `lessons_collected` / `lessons_closed` — 闭环率
- `auto_patches_applied` — 自动修复数
- `regressions` — 回归缺陷数
- `p0_lessons` — P0 级教训数
- `mean_time_to_fix_p0_minutes` — P0 平均修复时间

solo-audit Agent 1 读取此文件: 闭环率 <50% → `[EVOLVE_STAGNATION]`

## expansion-scan.py

路径: `scripts/expansion-scan.py`

检测规则:
- 技能文件周增长 >20% → 膨胀预警
- 新增规则关键词密度 vs LEARNED PATTERNS 比例异常
- 超大文件 (>800行) + 高规则密度

solo-audit Agent 1 运行此脚本，读取 `memory/.mev/expansion-scan.json`

## 特权环合规审计

solo-audit Agent 4 (ANALYZE) 铁律④子审计:
- 检查过去24小时 `skill_manage` 操作
- 读取 `_auto_fix_log.json`
- 验证 auto-fix 仅修改 `agents/*.md`
- 输出 `ring_compliance: { violations, total_ops, compliant }`
