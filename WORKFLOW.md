# Long-Running Coding Agent Workflow

This workflow is for unattended, multi-PR feature work. It assumes an orchestrator agent can run local commands, edit files, use `git`, use `gh`, invoke a separate reviewer agent, and run the project test suite.

## Principles

- No human is consulted mid-loop. If the agent cannot converge, it records the failure and stops.
- The reviewer is fresh every time. It sees only the files written to disk and the prompt.
- The state file is the source of truth. Update it after every phase boundary and reviewer call.
- Companion skills or modules are wrapped by this loop. Do not modify them; make them autonomous by converting their questions into decision records, conservative recommendations, or blockers.
- Internet verification is mandatory for plan review, decomposition review when external claims are involved, and per-subissue plan review.
- Every PR has a test contract before implementation begins.
- Iteration caps prevent infinite loops.
- Merge each PR as soon as it is clean.

## Companion Skill Adapters

The workflow can compose with pre-existing skills, modules, or documented sub-workflows:

- **`grill-my-plan` adapter**: run before plan-review checkpoints. Use its local-code inspection, external evidence, scenario pressure testing, and trade-off calibration to produce a written grill report. In unattended mode, any question it would normally ask a human becomes a conservative autonomous decision or a blocker.
- **`review-checkpoint` adapter**: run inside every implementation PR. It owns the locked test contract, incremental checkpoints, cumulative self-review, and final contract walk-through.

These adapters are optional but recommended. The long-running loop controls autonomy, state, reviewer calls, iteration caps, PR ordering, and E2E validation.

## State File

Use `/tmp/agent-loop-state.json`. It is scratch state and should not be committed. Start from [templates/state.json](templates/state.json).

Update it when:

- A phase starts or finishes
- A reviewer critique is produced
- A blocker or major finding is fixed
- A branch, issue, or PR is created
- Local tests pass or fail
- A PR is merged
- The loop aborts

## Reviewer Invocation Pattern

Every checkpoint follows the same pattern:

1. Write the artifact under review to disk.
2. Write any necessary context files to disk.
3. Update `/tmp/agent-loop-state.json`.
4. Invoke the reviewer agent with the relevant prompt.
5. Parse the critique for `**Severity: blocker**`, `**Severity: major**`, and `**Severity: minor**`.

Decision rule:

- Any blocker: fix every blocker and re-review.
- Three or more majors: fix the top three and re-review.
- Only minors: apply best-effort fixes and proceed.
- Iteration cap reached: mark the current phase as aborted, write `abort_reason`, and stop.

Default reviewer command:

```bash
claude --dangerously-skip-permissions -p "$(cat <<'EOF'
{prompt body}
EOF
)"
```

Resolve the reviewer in this order:

1. If `$AGENT_REVIEWER_CMD` is set, use it exactly.
2. Else, if `claude` exists on `PATH`, use `claude --dangerously-skip-permissions -p`.
3. Else, if the host exposes a fresh-agent or subagent mechanism, spawn a review-only agent with the prompt body and file paths.
4. Else, abort the current phase with `abort_reason = "No reviewer adapter available"`.

## Phase 1: Review The Plan

Iteration cap: 3.

1. Create a scratch branch named `agent/plan-{timestamp}`.
2. Save the user-provided plan verbatim to `plans/raw-plan.md`.
3. Initialize `/tmp/agent-loop-state.json`.
4. Run a non-interactive `grill-my-plan` pass and write `plans/grill-review-{N}.md`.
5. Run the plan review prompt in [prompts/phase-1-plan-review.md](prompts/phase-1-plan-review.md).
6. Copy `plans/raw-plan.md` to `plans/reviewed-plan.md` and apply blocker and major findings.
7. Re-review `plans/reviewed-plan.md` until there are no blockers and fewer than three majors.
8. Commit the reviewed plan:

```bash
git add plans/
git commit -m "plan: reviewed plan for {task_title}"
```

Abort if the third review still has blockers.

## Phase 2: Decompose Into Subissues

Iteration cap: 2.

Write `plans/subissues.md` using [templates/subissue.md](templates/subissue.md). Each subissue must:

- Produce one mergeable PR
- Have one concrete acceptance criterion
- Be verifiable in isolation
- Declare all dependencies explicitly
- Be small or medium sized

Run the decomposition review prompt in [prompts/phase-2-subissue-review.md](prompts/phase-2-subissue-review.md). Fix blockers and majors until the dependency graph is accepted.

Create tracker issues:

```bash
gh issue create \
  --title "{title}" \
  --body "$(cat subissue-{id}-body.md)" \
  --label "agent-loop"
```

Capture issue numbers and URLs in the state file. The resulting order is locked. If implementation proves the order is wrong, return to Phase 2 and re-review the dependency graph.

## Phase 3: Implement Each Subissue

Plan review cap: 2. PR review cap: 3.

For each subissue in dependency order:

1. Assign the issue and create a branch.

```bash
gh issue edit {issue_number} --add-assignee @me
git checkout main
git pull
git checkout -b {branch_name}
```

2. Write a test contract at `testing/{branchname}.md` using [templates/test-contract.md](templates/test-contract.md) and the `review-checkpoint` contract requirements.
3. Run a non-interactive `grill-my-plan` pass and write `testing/{branchname}-grill.md`.
4. Run the per-subissue plan review prompt in [prompts/phase-3-subissue-plan-review.md](prompts/phase-3-subissue-plan-review.md).
5. Fix blockers before coding.
6. Implement only the reviewed subissue scope using the `review-checkpoint` adapter.
7. Run unit, integration, smoke, manual, and E2E checks from the test contract.
8. Create a PR.

```bash
gh pr create \
  --title "{subissue title}" \
  --body-file /tmp/pr-body.md
```

9. Save the diff and run the PR review prompt in [prompts/phase-3-pr-review.md](prompts/phase-3-pr-review.md).

```bash
gh pr diff {pr_number} > /tmp/pr-{pr_number}.diff
```

10. Capture local verification output in `testing/{branchname}-local-test-log.md`.
11. Merge when the PR review has no blockers and local verification passes.

```bash
gh pr merge {pr_number} --squash --delete-branch
```

## Phase 4: End-To-End Validation

After every planned subissue is merged, set `state.e2e.status` to `in_progress`.

For API work:

- Start the service locally.
- Run every documented curl example.
- Verify status codes, response shapes, side effects, and logs.
- Write `e2e/api-test-log.md`.

For CLI work:

- Build or install the CLI locally.
- Run every documented invocation.
- Verify exit codes, stdout, stderr, and side effects.
- Write `e2e/cli-test-log.md`.

For UI work:

- Launch the dev server.
- Drive every documented user flow with a browser automation tool.
- Verify visual states, navigation, form submissions, and network calls.
- Write `e2e/ui-test-log.md`.

For mixed work, run every relevant path.

Then run the E2E review prompt in [prompts/phase-4-e2e-review.md](prompts/phase-4-e2e-review.md). If there are no blockers, set `state.e2e.status` to `passed`.

## Phase 5: E2E Bug-Fix Loop

If E2E review finds blockers or majors:

1. Open a fix issue for each blocker.

```bash
gh issue create \
  --title "fix: {short description from e2e review}" \
  --body "Found in E2E validation. See e2e/review.md." \
  --label "agent-loop,e2e-fix"
```

2. Treat each fix as a single subissue and run Phase 3.
3. Append each merged fix PR to `state.e2e.followup_prs`.
4. Re-run Phase 4.

Abort if more than five E2E follow-up PRs are needed.

## Abort Conditions

Abort cleanly when:

- Plan review hits its cap with unresolved blockers.
- Subissue decomposition hits its cap with unresolved blockers.
- A per-subissue plan review hits its cap with unresolved blockers.
- A PR review hits its cap with unresolved blockers.
- E2E follow-up PRs exceed five.
- `gh`, `git`, the reviewer adapter, companion skill adapter, or local test commands repeatedly fail with a non-recoverable error.

On abort:

- Update `/tmp/agent-loop-state.json`.
- Preserve branches and PRs.
- Leave logs and critique files inspectable.
- Stop without asking for human input.
