# Long-Running Coding Agent

A Ralph-loop-adjacent operating system for unattended, multi-PR coding agents.

Ship large coding-agent work without letting the agent drift.

This repo describes how to take a raw feature plan, verify it against current sources, split it into ordered subissues, implement each subissue through fresh reviewer checkpoints, merge incrementally, and finish with end-to-end validation. It is meant for coding agents that can run shell commands, create branches and pull requests, call a separate reviewer agent, and execute local tests.

The core idea is simple: the implementation agent should not trust its own plan forever. It keeps a state file, asks a fresh reviewer agent to critique every major artifact, fixes blockers, and stops cleanly if it cannot converge.

**Keywords:** AI coding agent, agentic coding, autonomous coding loop, Ralph loop, Claude Code, Codex, GitHub PR automation, spec-driven development, multi-agent software engineering, E2E testing.

## Why This Exists

Coding agents are getting good enough to work for hours, but the failure mode is still familiar: they start from a vague spec, invent stale APIs, skip hidden integration work, pass shallow tests, and call the job done.

This workflow is for the messy middle between a single prompt and a full platform:

- **Spec-first**: review the plan before code exists.
- **Plan grilling**: pressure-test the plan before the reviewer sees it.
- **Multi-PR by default**: split large work into dependency-ordered subissues.
- **Fresh reviewer loops**: use an independent reviewer context at every checkpoint.
- **Current-source verification**: check external APIs, library behavior, and service contracts before relying on them.
- **E2E or abort**: finish by exercising the user-visible workflow, or leave a clean failure trail.

## Relationship To Ralph Loops

This is intentionally close in spirit to the Ralph loop family of ideas: autonomous coding work should move through explicit phases like plan, implement, test, verify, and PR instead of one giant retry loop. [Wiggum's Ralph loop writeup](https://wiggum.app/blog/what-is-the-ralph-loop/) describes the value of phase isolation and explicit verification; [Ralph Loops](https://ralphloops.io/) packages that kind of loop as a portable directory with a `RALPH.md` entrypoint.

This repo is **not** a Ralph Loop package and does not require a Ralph runtime. It is a plain, tool-agnostic workflow for long-running coding agents. Think of it as:

- Ralph-loop adjacent
- More opinionated about multi-PR decomposition
- More aggressive about fresh reviewer checkpoints
- Focused on end-to-end validation after all PRs land
- Easy to adapt to Claude Code, Codex, OpenHands, Aider, Goose, or your own agent harness

## Optional Companion Skills

If your coding environment has reusable skills, modules, or sub-workflows, this repo treats them as adapters around the main loop. The two intended companions are:

- **`grill-my-plan`**: use it before plan-review checkpoints to pressure-test architecture, hidden subtasks, stale API assumptions, rollback gaps, and test coverage.
- **`review-checkpoint`**: use it inside each PR to lock the test contract, implement in small reviewed steps, and prove the final diff matches the contract.

The companion skills do not need to be changed. The long-running loop makes them autonomous by wrapping their outputs:

- If `grill-my-plan` would normally ask a human a question, route that question to the autonomous reviewer adapter. The resolver must answer it, narrow the scope, split out a generated follow-up/subissue, or choose the safest reversible default.
- If `review-checkpoint` needs more contract detail, update the subissue test contract and re-run the plan reviewer before coding.
- A grill question is not a blocker by default. Only abort when every available autonomous action would be unsafe, destructive, or unverifiable.

## When To Use It

Use this loop when the work is:

- Large enough to require multiple PRs, or one PR that needs internal staging
- Expected to run unattended for a long time
- Verifiable end to end through an API, CLI, UI, or mixed workflow
- Risky enough that drift, stale API assumptions, or shallow tests would be expensive

Do not use it for quick bug fixes, single-file edits, or exploratory research. The overhead earns its keep only when the agent is shipping real surface area.

## The Model

There are two roles:

- **Orchestrator agent**: plans, edits code, opens issues, opens PRs, merges, and runs E2E.
- **Reviewer agent**: starts from a fresh context at each checkpoint and writes a structured critique. It does not write implementation code.

The default reviewer invocation in this repo uses Claude Code:

```bash
claude --dangerously-skip-permissions -p "<prompt>"
```

If Claude Code is not installed, use `$AGENT_REVIEWER_CMD` or the host's own fresh-agent/subagent mechanism. The reviewer only needs to read files, use the internet when needed, and write markdown critique files. It must not write implementation code.

## Workflow

1. **Plan review**: Save the raw plan, verify external claims against current sources, and rewrite the plan until it has no blockers.
2. **Subissue decomposition**: Split the reviewed plan into ordered, testable, independently mergeable issues.
3. **Per-subissue loop**: For each issue, write and review a test contract, implement against it, review the PR diff, run local verification, and merge.
4. **End-to-end validation**: After every subissue is merged, exercise the whole feature from the outside.
5. **Bug-fix loop**: Convert E2E blockers into follow-up issues and run the same per-subissue loop again.

The full operating spec is in [WORKFLOW.md](WORKFLOW.md).

## Repo Contents

- [WORKFLOW.md](WORKFLOW.md): complete long-running agent loop
- [templates/state.json](templates/state.json): shared state file shape
- [templates/grill-review.md](templates/grill-review.md): autonomous plan-grilling report format
- [templates/grill-question-resolution.md](templates/grill-question-resolution.md): autonomous resolver output format
- [templates/subissue.md](templates/subissue.md): subissue format
- [templates/test-contract.md](templates/test-contract.md): per-PR test contract format
- [prompts/](prompts): reviewer prompt templates

## Safety Rails

- Keep `/tmp/agent-loop-state.json` as the single source of truth.
- Use a fresh reviewer context at every checkpoint.
- Prefer `$AGENT_REVIEWER_CMD`, then Claude Code, then a fresh self-spawned reviewer if the host allows it.
- Verify external APIs and service contracts with current sources before coding.
- Merge clean PRs eagerly instead of batching them.
- Abort cleanly when iteration caps are reached.
- Never silently reorder subissues; re-review the dependency graph first.

## Discovery Terms

If you are searching for this pattern, these are the phrases people use around it:

- autonomous coding loop
- Ralph loop
- long-running coding agent
- agentic coding workflow
- AI coding agent PR loop
- spec-driven coding agents
- multi-agent software engineering
- E2E validation for coding agents
- fresh-context reviewer agent

These terms line up with how the space is being indexed right now: OSSInsight tracks categories like AI agents, coding agents, MCP servers, vibe coding, and AI assistants; recent coding-agent research uses terms like agentic coding, agentic PRs, and agentic software engineering.

## License

MIT. See [LICENSE](LICENSE).
