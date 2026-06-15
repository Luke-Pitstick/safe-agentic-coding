---
name: decompose-task
description: Split large, vague, or high-level goals into agent-ready subtasks with clear scope, context, prompts, dependencies, acceptance criteria, expansion paths, and reuse/library research where existing tools could reduce complexity. Use when the user asks to break down a project, plan work for multiple agents, create sub-tasks, turn an abstract idea into executable work, produce tickets from a broad goal, or prepare a delegation plan that other agents can run and elaborate.
---

# Decompose Task

## Overview

Turn broad intent into a small portfolio of executable task cards. Each task card must be clear enough for another agent to start, investigate, expand, and complete without relying on hidden conversation context.

Use `$delegate-agent-tasks` after this skill when the user wants Codex to spawn subagents that execute the task cards and integrate their results.

## Optional GStack and GBrain Compatibility

Use GStack and gbrain as optional memory, never as a required dependency.

Before decomposing, if `gbrain` is on PATH:

- Extract 2-4 concrete keywords from the goal, such as feature names, error names, module names, or product areas.
- Run `gbrain search "<keywords>"`.
- Read at most the top 3 clearly relevant pages with `gbrain get_page "<slug>"`.
- Use only source-linked context that helps scope tasks, identify prior decisions, or avoid repeated work.
- If `gbrain` is unavailable, returns an error, or has no useful hits, continue from local workspace files.

After writing the decomposition, save a compact summary when `gbrain` is available:

```bash
gbrain put "safe-agentic/tasks/<goal-slug>" --content "<markdown summary>"
```

The saved summary should include the goal, task-card path, critical path, major dependencies, and validation gates. Do not save secrets, credentials, raw user payloads, private keys, sensitive PII, or large code dumps. The project-local `agents/` artifact remains the source of truth.

## Operating Rules

- Preserve the user's strategic intent before optimizing the work breakdown.
- Read relevant local artifacts before decomposing when the task depends on a repo, document, plan, design, issue, log, or dataset.
- Ask only for missing information that changes the decomposition materially. Otherwise state assumptions and proceed.
- Prefer 3-9 meaningful subtasks. Split further only when a task has multiple owners, different acceptance criteria, or a dependency boundary.
- Make each subtask independently runnable. Include the exact inputs, starting context, expected outputs, and validation method.
- Do not turn uncertainty into vague TODOs. Convert uncertainty into discovery steps, decision points, or explicit open questions.
- Keep sequencing visible. Call out what can run in parallel, what must wait, and what can be deferred.
- Write the decomposition to the current workspace's `agents/` folder by default, unless the user explicitly asks for chat-only output or another path.
- When the goal involves custom implementation of common capabilities, include a `$deep-dive`-driven reuse/library investigation or add a reuse check inside the relevant task card.

## Workflow

### Step 1: Frame the Goal

Summarize the high-level task in 3-5 bullets:

- Desired outcome
- Known constraints
- Stakeholders or users affected
- Existing artifacts inspected
- Assumptions made

If the goal is ambiguous enough that any decomposition would be misleading, stop and ask one concise question. High-stakes ambiguity includes unclear production targets, destructive operations, legal or financial consequences, or incompatible success criteria.

### Step 2: Map the Work

Create a decomposition map before writing task cards:

- Identify major workstreams: research, design, architecture, implementation, migration, QA, documentation, release, operations.
- Identify dependencies and integration points.
- Identify risk areas that need early investigation.
- Identify areas where an existing library, framework feature, standard-library API, or existing project module could reduce implementation complexity.
- Decide which work can be parallelized.
- Decide the minimum complete slice that proves the plan works.

Use evidence from the user's artifacts when possible. If no artifacts exist, make the map from the user's stated goal and label it as assumption-based.

### Step 3: Shape Subtasks

For each subtask, define:

- Outcome: the concrete result this task produces.
- Scope: what is included and explicitly out of scope.
- Context packet: files, docs, links, commands, decisions, and constraints the agent needs.
- Agent instructions: the prompt another agent should follow.
- Expansion path: how the agent should deepen the idea, discover edge cases, or split child tasks.
- Reuse/library check: whether the agent should use `$deep-dive` to compare existing libraries, framework APIs, or reusable modules before building custom code.
- Acceptance criteria: observable checks that prove completion.
- Validation: tests, review steps, demos, screenshots, metrics, or manual checks.
- Dependencies: prerequisites and downstream consumers.
- Handoff: the artifact the agent should return.

Read `references/task-card-template.md` when the user asks for issue-ready, agent-prompt-ready, or copy-pastable output.

### Step 4: Sequence the Portfolio

Group subtasks into execution lanes:

- Now: tasks that unblock the rest of the work.
- Parallel: tasks that can run concurrently after the first unblockers.
- Integration: tasks that combine outputs from multiple agents.
- Later: useful follow-ups that should not block the core outcome.

Name the critical path and the riskiest assumption. If the split creates coordination overhead larger than the work itself, merge tasks and explain why.

### Step 5: Write the Plan

Write the full decomposition as a Markdown file in the current workspace:

- Resolve the workspace root with `git rev-parse --show-toplevel` when inside a Git repo; otherwise use the current working directory.
- Use `<workspace-root>/agents/` as the default destination.
- Create `agents/` if it does not exist.
- Name the file after the task instead of using a generic `subtasks.md` filename.
- Derive a concise lowercase slug from the goal, using hyphens for spaces and removing punctuation, then write `agents/<goal-slug>-subtasks.md`.
- Keep the slug specific enough to recognize the task later, such as `checkout-refactor-subtasks.md`, `billing-retry-qa-subtasks.md`, or `mobile-onboarding-redesign-subtasks.md`.
- If the derived filename already exists, inspect it. Append a short disambiguator such as `-v2`, a subsystem name, or a date only when the existing file is for a different or stale plan.
- If the user names a specific file, use that path instead.
- If the user asks for no files, emit the plan in chat only.

Default output structure:

```markdown
# <Goal Name> Subtasks

Generated: <YYYY-MM-DD>

## Goal
[Short goal summary]

## Assumptions
- [Assumption]

## Execution Shape
- Critical path: [T1 -> T3 -> T7]
- Parallel lanes: [T2, T4, T5]
- Integration point: [where outputs converge]

## Subtasks
### T1: [Verb-led title]
[Task card]

## Coordination Notes
- [Dependency, risk, or decision gate]

## Suggested Next Dispatch
[The first agent prompt to run]
```

Keep each task card concise but complete. Prefer precise verbs such as audit, design, implement, migrate, test, document, validate, compare, or decide.

When a subtask asks an agent to build common infrastructure, add a line such as: `Before implementing custom code, use $deep-dive to check for existing libraries, framework APIs, or standard-library features that meet the need; adopt only if they reduce complexity after integration cost.`

After writing the file, summarize the path and the critical path in chat. Do not paste the entire file unless the user asks.

If the user asks to execute the plan with agents, invoke `$delegate-agent-tasks` with the emitted task cards rather than spawning subagents from this skill directly.

## Quality Bar

Before finalizing, check:

- Every task has a clear owner shape: one agent can run it.
- Every task has a deliverable and acceptance criteria.
- The Markdown plan has been written under `agents/` or the user explicitly requested chat-only output.
- No task depends on context that is only in your head.
- Dependencies are explicit and acyclic unless there is an intentional iteration loop.
- The first task reduces uncertainty or creates momentum.
- Common infrastructure tasks include an explicit build-vs-adopt check when relevant.
- The plan leaves room for agents to expand ideas without drifting from the goal.

## Anti-Patterns

- Do not output a flat checklist with no context packets.
- Do not create tiny tasks that require constant synchronization.
- Do not bury decision gates inside implementation tasks.
- Do not assign parallel agents to edit the same files or artifacts without a merge plan.
- Do not use "research this" as a complete subtask. Specify what to inspect, what to compare, and what conclusion to return.
