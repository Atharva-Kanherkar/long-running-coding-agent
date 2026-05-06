# Phase 4: E2E Review Prompt

Read `e2e/*.md`, `plans/reviewed-plan.md`, `plans/subissues.md`, and `/tmp/agent-loop-state.json`.

Verify:

1. Every acceptance criterion in `plans/subissues.md` was exercised in the E2E logs.
2. The full happy path works across all merged subissues.
3. Error paths return correct statuses, shapes, and side effects.
4. Logs show no silent failures, swallowed errors, or unexpected warnings.

Output `e2e/review.md` with severity-tagged findings.

Do not modify code or tests.
