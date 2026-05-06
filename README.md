# Long-Running Coding Agent

A battle-tested operating loop for unattended, multi-PR coding agents.

This repo describes how to take a raw feature plan, verify it, split it into ordered subissues, implement each subissue through review checkpoints, merge incrementally, and finish with end-to-end validation. It is meant for coding agents that can run shell commands, create branches and pull requests, call a separate reviewer agent, and execute local tests.

The core idea is simple: the implementation agent should not trust its own plan forever. It keeps a state file, asks a fresh reviewer agent to critique every major artifact, fixes blockers, and stops cleanly if it cannot converge.

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

You can adapt the prompts for any reviewer agent that can read files, use the internet when needed, and write markdown critique files.

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
- [templates/subissue.md](templates/subissue.md): subissue format
- [templates/test-contract.md](templates/test-contract.md): per-PR test contract format
- [prompts/](prompts): reviewer prompt templates

## Safety Rails

- Keep `/tmp/agent-loop-state.json` as the single source of truth.
- Use a fresh reviewer context at every checkpoint.
- Verify external APIs and service contracts with current sources before coding.
- Merge clean PRs eagerly instead of batching them.
- Abort cleanly when iteration caps are reached.
- Never silently reorder subissues; re-review the dependency graph first.

## License

MIT. See [LICENSE](LICENSE).
