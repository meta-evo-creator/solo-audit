# Fleet Health Check Methodology

> 2026-07-21 retrospective: the SOUL.md solo fleet table was inconsistent with the skills/ directory.

## Problem

This check found fleet drift:

| Direction | Type | Finding |
|:-----|:-----|:------|
| Table → disk | ❌ dead link | `party-review`, `auto` — present in the table, absent from disk |
| Disk → table | ❌ omission | `legal-db-search`, `tyc-it` — present on disk, absent from the table |
| Disk → table | ❌ partial omission | `zhihu-search` — present in the matching table, absent from the three-axis cluster list |

## Root cause

Only **one-directional verification** (table → disk) was performed, not **reverse verification** (disk → table). One-directional checking finds dead links but cannot find valid skills that were never registered.

## Correct procedure

```
1. List all directories under skills/
2. List all skill names in the three-axis clusters of SOUL.md
3. Compare both directions:
   - In the table but not on disk → dead link, must be deleted or repaired
   - On disk but not in the table → unregistered, must be added
4. The skill matching table must also be updated in sync
```

## Trigger conditions

- After adding/removing a skill
- After modifying SOUL.md
- Periodic audit
- When a skill's `skill_view` reports unsupported (possibly a registration problem)
