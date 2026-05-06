# Phase 2: Subissue Review Prompt

Read `plans/reviewed-plan.md`, `plans/subissues.md`, and `/tmp/agent-loop-state.json`.

Validate:

1. Coverage: does the union of subissues fully implement the plan?
2. Ordering: is the `depends_on` graph correct? Identify missing dependencies and cycles.
3. Sizing: is any subissue actually multiple PRs in disguise?
4. Testability: can each acceptance criterion be verified in isolation?
5. Independence: does any subissue introduce coupling that will force the order to change later?

Output `plans/subissue-review-{N}.md` with this structure:

```markdown
# Subissue Review {N}

## Coverage
- **Severity: blocker|major|minor** - Finding - Recommended fix

## Ordering
- **Severity: blocker|major|minor** - Finding - Recommended fix

## Sizing
- **Severity: blocker|major|minor** - Finding - Recommended fix

## Testability
- **Severity: blocker|major|minor** - Finding - Recommended fix

## Recommended Changes
- Concrete edit to make
```

Do not implement anything.
