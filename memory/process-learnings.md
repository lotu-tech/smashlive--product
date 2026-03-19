# Process Learnings

## Developer must lint/format before PR
**Issue:** Reviewer rejected PRs for lint/format issues, wasting full agent cycles.
**Fix:** Developer prompt now requires running lint + format + commit before ADVANCE.
**Date:** 2026-03-19

## Reviewer should not reject for lint/format only
**Issue:** Reviewer was rejecting for trivially fixable formatting issues.
**Fix:** Reviewer prompt now says lint/format is NOT a rejection reason — ADVANCE with a comment instead.
**Date:** 2026-03-19

## Scoper must assign project labels
**Issue:** Issues created by PM approval had no project: labels. Orchestrator skipped them.
**Fix:** Scoper prompt now reads the spec and assigns labels. Approval flow always goes to Scoping column.
**Date:** 2026-03-19

