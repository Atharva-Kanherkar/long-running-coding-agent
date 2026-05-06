# Phase 1: Plan Review Prompt

You are reviewing a coding plan before any code is written. The plan is at `plans/raw-plan.md` or `plans/reviewed-plan.md`. The current state of the agent loop is in `/tmp/agent-loop-state.json`. Also read the latest `plans/grill-review-*.md` if it exists.

Your job:

1. Verify every external claim against current sources. Library APIs, framework behavior, version compatibility, third-party service contracts, RFC details, and platform constraints must be checked with internet sources. Cite source URLs.
2. Find architectural problems: race conditions, missing error paths, unclear ownership, security holes, observability gaps, migration risks, and rollback gaps.
3. Identify hidden subtasks that need their own design.
4. Assess whether the plan is implementable as described.
5. Check whether the grill report converted unresolved human questions into safe autonomous decisions or legitimate blockers.

Output your critique to `plans/plan-review-{N}.md`, where `{N}` is the next available integer. Use this structure:

```markdown
# Plan Review {N}

## Verified Claims
- Claim - Source URL - Notes

## Refuted Or Stale Claims
- Claim - Source URL - Correct fact

## Architectural Concerns
- **Severity: blocker** - Description - Recommended fix
- **Severity: major** - Description - Recommended fix
- **Severity: minor** - Description - Recommended fix

## Missing Subtasks
- Subtask - Why it is needed

## Recommended Changes
- Concrete edit to make
```

Do not write implementation code. Do not modify any file other than the critique.
