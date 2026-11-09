# Cron Modification Guardrail

## Iron rule
When fixing cron infrastructure (paths / times / keys), **you must not modify the content format at the same time**.

## Case
On 2026-07-21, while fixing the Hermes paths of three report crons, the report template was modified at the same time — changed from the original intelligence briefing format into a governance document containing a MEV matrix, DELIVERY CHECK, and bias self-check.

## Correct approach
- Fix paths: change the absolute path, search tool, workdir
- Fix times: change the schedule
- Fix content: raise a separate PR, do not mix it into infrastructure fixes

## Verification
After changing a cron → immediately run `cronjob run` → check whether the output format matches the previous day's
