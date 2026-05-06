# Phase 3: Per-Subissue Plan Review Prompt

Read `testing/{branchname}.md`, `testing/{branchname}-grill.md` if it exists, `plans/reviewed-plan.md`, `plans/subissues.md`, and `/tmp/agent-loop-state.json`.

Verify:

1. The plan implements exactly the selected subissue scope.
2. Every external API or library reference is current. Use the internet and cite source URLs.
3. The test sections catch likely failure modes.
4. The plan does not depend on unmerged subissues.
5. The rollback strategy is real.
6. The autonomous grill decisions are safe, or correctly marked as blockers.

Output `testing/{branchname}-review-{N}.md` with severity-tagged findings.

Do not implement anything.
