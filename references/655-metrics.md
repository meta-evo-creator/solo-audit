# 655 Evolution Metrics Baseline

## evolution-metrics.json

Path: `memory/.mev/evolution-metrics.json`

Tracked metrics:
- `lessons_collected` / `lessons_closed` — closure rate
- `auto_patches_applied` — number of automatic fixes
- `regressions` — number of regression defects
- `p0_lessons` — number of P0-level lessons
- `mean_time_to_fix_p0_minutes` — mean time to fix P0

solo-audit Agent 1 reads this file: closure rate <50% → `[EVOLVE_STAGNATION]`

## expansion-scan.py

Path: `scripts/expansion-scan.py`

Detection rules:
- Weekly growth of skill files >20% → expansion warning
- Anomalous ratio of new-rule keyword density against LEARNED PATTERNS
- Oversized files (>800 lines) + high rule density

solo-audit Agent 1 runs this script and reads `memory/.mev/expansion-scan.json`

## Privileged-ring compliance audit

solo-audit Agent 4 (ANALYZE) Iron Rule 4 sub-audit:
- Check `skill_manage` operations over the past 24 hours
- Read `_auto_fix_log.json`
- Verify that auto-fix only modified `agents/*.md`
- Output `ring_compliance: { violations, total_ops, compliant }`
