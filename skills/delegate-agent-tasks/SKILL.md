---
name: delegate-agent-tasks
description: Delegate agent-ready tasks to Codex subagents and coordinate execution, validation, GStack review gates, QA, design review, and final integration. Use when the user asks Codex to delegate work, spawn subagents, run subtasks, parallelize a plan, execute task cards from decompose-task, run GStack review skills, run qa/design-review/review gates, coordinate multiple agents, or integrate outputs from delegated agent work.
---

# Delegate Agent Tasks

## Overview

Execute a task-card portfolio by spawning Codex subagents for independent work and integrating their results. Use this after a decomposition exists, especially from `$decompose-task`, or when the user already provides task cards.

## Optional GStack and GBrain Compatibility

Use GStack and gbrain as optional coordination memory, never as a required dependency.

Before dispatch, if `gbrain` is on PATH:

- Extract 2-4 concrete keywords from the task portfolio, such as task IDs, feature names, module names, or bug names.
- Run `gbrain search "<keywords>"`.
- Read at most the top 3 clearly relevant pages with `gbrain get_page "<slug>"`.
- Attach relevant memory findings to subagent prompts only when they reduce repeated work, clarify prior decisions, or reveal known risks.
- If `gbrain` is unavailable, returns an error, or has no useful hits, continue from task cards and local artifacts.

After integration, save a compact handoff when `gbrain` is available:

```bash
gbrain put "safe-agentic/handoffs/<task-or-goal-slug>" --content "<markdown summary>"
```

The saved summary should include completed task IDs, artifact paths, commits created, validation gates run, skipped gates with reasons, and remaining risks. Do not save secrets, credentials, raw user payloads, private keys, sensitive PII, or large diffs. The project-local `agents/` artifacts and Git history remain the source of truth.

## Tool Setup

Use Codex subagent tools when available:

- `multi_agent_v1.spawn_agent` to create a subagent.
- `multi_agent_v1.wait_agent` to wait for required results.
- `multi_agent_v1.send_input` to clarify or redirect a running agent.
- `multi_agent_v1.close_agent` to close agents that are no longer needed.

If these tools are not loaded, use `tool_search` to search for `spawn_agent`, `multi-agent`, or `subagent`. If no subagent tool is available, do not pretend to dispatch work. Produce a manual dispatch plan with copy-pastable prompts instead.

Invocation of this skill is explicit permission to use subagents for the task, unless the user says not to.

## Dispatch Rules

- Keep the critical-path blocker local when your next step depends on it.
- Spawn subagents only for concrete, bounded subtasks that materially advance the goal.
- Prefer parallel delegation for independent research, audits, verification, or disjoint code changes.
- Allow implementation subagents to write code when the task requires it and they have a clear, bounded write scope.
- Do not spawn multiple agents into the same files, modules, docs, or design artifacts without a merge plan.
- Give every subagent a complete task card. Do not rely on hidden conversation context.
- Tell code-editing agents they are not alone in the codebase and must not revert unrelated changes.
- Commit regularly when working in a Git repository: after a coherent subtask is complete, validated, and scoped to owned files.
- Wait only when the result is needed for integration or final reporting.
- Poll running agents in a coordination loop instead of interrupting them for routine status.
- Require every worker to emit a clear final handoff message when done, blocked, failed, or waiting on another subagent.
- Review returned work before treating it as complete.
- Run relevant review, QA, and design-review skills as gates around delegated work when those skills are available.
- Close agents when their output has been integrated or no longer matters.

## Skill Routing Gates

Use relevant skills as execution and review gates. If a named skill is not available in the current session, do not invent it; note that the gate was skipped or use the nearest available equivalent.

### Before Dispatch

- Use `$decompose-task` when the work has no task cards yet.
- Use `$autoplan` for a broad plan that needs CEO, design, engineering, and DX review before work starts.
- Use `$plan-ceo-review` for strategy, scope, product premise, or "should we do this?" uncertainty.
- Use `$plan-eng-review` for architecture, data flow, dependency, migration, performance, or testing-plan risk.
- Use `$plan-design-review` for UI/UX plans, design systems, visual hierarchy, or interaction models.
- Use `$plan-devex-review` for developer-facing APIs, CLIs, docs, SDKs, setup flows, or internal tooling.

### During Delegation

- Use `$write-tests` for subagents whose main output is test coverage or regression protection.
- Use `$investigate` for bug/root-cause subtasks before assigning implementation.
- Use `$design-consultation` for design exploration before assigning UI implementation.
- Use `$cso` or security-review skills for auth, secrets, permissions, payments, infrastructure, or supply-chain-sensitive tasks.
- Use `$document-generate` or `$document-release` for documentation subtasks when those skills are available.

### After Integration

- Use `$review` for code diffs before considering delegated implementation complete.
- Use `$qa` for web/app behavior when the task includes a runnable product and fixes should be attempted.
- Use `$qa-only` when the user wants a report without source changes.
- Use `$design-review` for UI, visual polish, hierarchy, spacing, accessibility, responsive behavior, or AI-slop checks.
- Use `$canary` after deployment or production-like release verification.

Choose validation gates based on the task:

- Code changes: `$review` plus the repository's relevant test/type/lint command.
- Test-writing tasks: `$write-tests` during implementation, then run the targeted tests the task added or changed.
- Web/app behavior: `$qa` when fixes are allowed, `$qa-only` when the user wants report-only validation.
- UI/visual changes: `$design-review` after implementation, and `$qa` when interaction behavior also matters.
- Plan-only or architecture work: `$plan-eng-review`, `$plan-design-review`, or `$autoplan`; do not run implementation QA unless code or runnable artifacts exist.
- Docs/devex changes: `$plan-devex-review` for plans, `$review` for docs diffs, and runnable smoke checks for examples or setup flows.
- Deployment/release verification: `$canary` only after a deployment or production-like target exists.

When multiple gates apply, prefer this order: plan review -> delegated execution -> code review -> automated tests -> QA/design review -> final integration report. Do not run validation gates that do not match the task surface.

## Workflow

### Step 1: Normalize the Task Cards

Gather or create the execution set:

- If the user provides task cards, parse them into task IDs.
- If the user provides only a broad goal, first use `$decompose-task` or create a small decomposition yourself.
- If task cards lack context packets, acceptance criteria, or handoff artifacts, repair them before spawning agents.

Read `references/subagent-dispatch-template.md` when you need a copy-pastable subagent prompt or dispatch board.

### Step 2: Choose the Local Work

Before spawning, decide what you will do locally right now:

- Pick the immediate blocker on the critical path.
- Pick integration, review, or coordination work that should remain with the parent agent.
- State the local task briefly before spawning subagents.

Do not hand off the parent agent's coordination responsibility.

### Step 3: Select Subagent Assignments

For each candidate task, decide:

- Agent type: use `worker` for production edits, `explorer` for specific codebase questions, and default for general execution.
- Context mode: use `fork_context: false` unless the subagent needs the current conversation. Prefer explicit task-card context over forked hidden context.
- Write ownership: specify exact files, directories, modules, docs, or "read-only".
- Write permission: grant write permission for implementation, test-writing, docs, migration, fixture, or artifact-creation subtasks when a bounded ownership scope exists. Use read-only for review, QA report, investigation-only, or plan-only subtasks.
- Commit strategy: assign whether the subagent should commit its own completed work or leave changes unstaged for parent integration.
- Skill routing: assign any relevant skills the subagent should use, such as `$write-tests`, `$investigate`, `$review`, `$qa`, `$qa-only`, or `$design-review`.
- Dependencies: spawn now only if prerequisites are satisfied.

Spawn agents in parallel only when their ownership and outputs are independent.

### Step 4: Spawn Agents

For each spawned agent, pass a full prompt that includes:

- Parent goal
- Task card
- Relevant gbrain context, if available and source-linked
- Owned files or read-only boundary
- Whether the agent may edit files, and exactly which files/directories/modules it owns if edits are allowed
- Commit instructions: when to commit, what message shape to use, and whether to leave changes unstaged instead
- Relevant skills to invoke, if any
- Required artifacts
- Validation expectations
- Completion signal and dependency rules
- Return format
- Warning not to revert unrelated changes

Use `references/subagent-dispatch-template.md` for the prompt structure.

Spawn review/QA/design subagents as separate read-only reviewers when useful:

- Code reviewer: use `$review` on the integrated diff or task-specific diff.
- QA reviewer: use `$qa` or `$qa-only` against the runnable app or target URL.
- Design reviewer: use `$design-review` against the changed UI or screenshots.
- Plan reviewer: use `$plan-eng-review`, `$plan-design-review`, or `$autoplan` before implementation starts.

Reviewer agents default to read-only. Grant write scope to a reviewer only when the user or task explicitly asks that agent to fix issues it finds, and keep the fix scope disjoint from other active agents.

### Step 4.5: Commit Checkpoints

Use commits as checkpoints when the workspace is a Git repository and the user has not asked to avoid commits:

- Commit after each coherent implementation subtask or integration milestone that passes its relevant targeted validation.
- Prefer one commit per accepted task card or tightly related group of task cards.
- Run `git status --short` before staging. Stage only files owned by the completed task.
- Do not stage unrelated user changes, unrelated agent changes, local env files, generated junk, or broad working-tree changes by default.
- Use concise commit messages such as `Implement T2 auth validation` or `Add tests for T4 import parser`.
- Include the task ID in the commit message when task cards have IDs.
- If multiple subagents are working in the same repository, prefer parent-agent commits after integration unless each subagent has a disjoint worktree or clearly disjoint file ownership.
- If validation fails, do not commit unless the commit intentionally captures a failing reproduction and the user/task asked for that.
- If no Git repository exists, skip commits and report that checkpointing was unavailable.

Subagents may commit their own work only when their prompt explicitly says so. Otherwise they should report changed files and leave integration/commit decisions to the parent agent.

### Step 5: Continue Local Work

After spawning, do useful non-overlapping work immediately:

- Resolve the critical-path local task.
- Prepare integration scaffolding.
- Inspect artifacts that the subagents will touch.
- Build a dispatch board with agent IDs, task IDs, status, ownership, and expected handoff.
- Track commit status for each task: not ready, ready to commit, committed, or left uncommitted with reason.

Do not redo delegated work unless the subagent fails or returns unusable output.

### Step 5.5: Poll Workers Without Interrupting

Run a lightweight coordination loop while any subagent is active:

1. Poll each running worker at regular intervals using the available wait/status primitive.
2. Update the dispatch board with: running, waiting-on-task, blocked, complete, partial, failed, or needs-parent-input.
3. Do not send routine "are you done?" messages. Let workers continue uninterrupted until they emit their final handoff or an explicit blocker message.
4. If a worker reports that it is blocked by another subagent, record the dependency in the dispatch board and keep polling both agents.
5. If a dependency unblocks, send the minimum necessary handoff artifact to the blocked worker and let it resume.
6. Interrupt with `send_input` only when the worker asks for parent input, is narrowly stuck, needs another worker's completed artifact, or returned a fixable incomplete result.
7. Prefer keeping dependent workers alive in a waiting state over closing and respawning them when they need another subagent's output.

Require workers to end with a final completion signal in their return message:

```text
FINAL HANDOFF
Task ID: <id>
Status: complete | partial | blocked | failed
Blocks/depends on: <none or task/agent/artifact>
Ready for integration: yes | no
```

The parent agent should treat this signal as the worker's done/blocked notification and should not need to interrupt the worker to discover normal completion.

### Step 6: Integrate Results

Wait for agents when their results are needed. For each returned result:

- Check changed files or produced artifacts.
- Compare output against the task card acceptance criteria.
- Run targeted validation when possible.
- Run applicable review gates from "Skill Routing Gates".
- Identify conflicts, duplicate work, or missing handoffs.
- Ask the same agent for a fix with `send_input` if the gap is narrow and context-specific.
- Send dependent handoff artifacts to waiting workers before integrating locally when that lets the original worker finish its assigned task.
- Fix integration issues locally when they cross task boundaries.
- Commit the accepted result when it is coherent, validated, and safe to stage without unrelated changes.

Close the agent after its output is accepted or superseded.

### Step 7: Final Report

Report:

- Subagents spawned and their task IDs.
- Review/QA/design gates run, including skipped gates and why.
- What you handled locally.
- Integrated outputs and changed artifacts.
- Commits created, or why accepted work was left uncommitted.
- Validation performed.
- Remaining risks, follow-up task cards, or failed dispatches.

If a subagent could not be spawned, say why and provide the manual prompt that would have been sent.

## Safety Gates

Stop and ask before spawning when a task would:

- Touch production systems, credentials, billing, or destructive operations.
- Require approvals the subagent cannot request safely.
- Modify the same write scope as another active agent.
- Depend on unclear success criteria where wrong execution would be costly.

## Quality Bar

Before finalizing, check:

- Every spawned agent had a bounded task and complete prompt.
- Every code-editing agent had disjoint ownership.
- Implementation agents were allowed to edit code only inside their assigned scope.
- Read-only review agents did not modify files unless explicitly granted fix scope.
- Completed coherent work was committed regularly, or skipped with an explicit reason.
- The parent agent retained integration responsibility.
- Every accepted result was reviewed against acceptance criteria.
- Relevant GStack review, QA, and design-review gates were run or explicitly skipped with a reason.
- Agents were closed when done.
- The final answer distinguishes completed work from spawned-but-failed or deferred work.
