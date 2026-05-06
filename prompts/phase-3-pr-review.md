# Phase 3: PR Review Prompt

Review `/tmp/pr-{pr_number}.diff` against `testing/{branchname}.md`. Also read `plans/reviewed-plan.md`, `plans/subissues.md`, and `/tmp/agent-loop-state.json`.

Check:

1. Every test contract item is satisfied by the diff.
2. No code in the diff is outside the contract.
3. Correctness, error handling, race conditions, resource leaks, and security issues.
4. External API calls match the verified signatures in the test contract.
5. Tests assert meaningful behavior.

Output `testing/{branchname}-pr-review-{N}.md` with severity-tagged findings.

Do not modify code.
