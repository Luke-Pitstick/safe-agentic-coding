# Safe Agentic Coding

Safe Agentic Coding is a small collection of agent skills designed to create reproducible, safe, and efficient agentic
codding workflows. It is designed to work around an agents folder created by one of the skills. The folder holds relevant
agent references, documentation, and run loops.

There are a lot of nice features that attempt to replicate features I've seen in other tools. My favorites are
`debug-runtime` which replicates the Cursor debug mode by creating temporary logs and backtraces, my other favorites are
`decompose-task` which is a highly customized version of a plan mode designed to work with the `delegate-agent-tasks`
skill.

Please feel free to use and adapt these skills for your own projects.

## Workflow


```text
create project -> expand/decompose work -> research or simplify -> delegate implementation -> debug with runtime evidence -> test/review -> clean up repo state
```

The skills use the Agent Skills folder format:

```text
skill-name/
├── SKILL.md
├── agents/openai.yaml
└── references/
```

Each `SKILL.md` contains YAML frontmatter with `name` and `description`, followed by the instructions an agent should
load when the skill matches a task.

## Included Skills

| Skill                  | Purpose                                                                                                                                                |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| `create-project`       | Existing project scaffold skill updated to create `agents/`, `docs/`, and an `AGENTS.md`                                                               |
| `decompose-task`       | Splits high-level goals into agent-ready subtasks written to `agents/`, including context packets, dependencies, validation, and reuse/library checks. |
| `delegate-agent-tasks` | Coordinates subagents to execute task cards, including code-writing permissions, validation gates, and regular commit checkpoints.                     |
| `write-tests`          | Designs and writes robust unit, integration, regression, contract, and workflow tests.                                                                 |
| `expand-task`          | Expands a simple prompt into a narrow, app-aware implementation brief for another agent.                                                               |
| `deep-dive`            | Performs focused web research with sourced findings for startup, product, company, market, and technical questions.                                    |
| `science-check`        | Tests ideas against scientific and technical evidence, separating claims, mechanisms, evidence quality, and uncertainty.                               |
| `debug-runtime`        | Debugs tricky bugs with temporary session-tagged instrumentation, runtime logs/backtraces, narrow fixes, verification, and cleanup.                    |
| `update-gitignore`     | Audits a repo, updates `.gitignore`, and untracks already-tracked ignored files without deleting local copies.                                         |
| `simplify-code`        | Analyzes code complexity and finds simpler behavior-preserving implementations, including library/API alternatives via `$deep-dive`.                   |
| `tech-discovery`       | Researches technologies, libraries, APIs, open source projects, standards, datasets, and platforms that can bootstrap a project idea.                  |
| `write-docstrings`     | Analyzes current code and writes docstrings in the format of the current language. Writes usage, parameters, and outputs.                              |

## Using `create-project`

Use `create-project` when you want an agent to start a new local project and matching GitHub repository from a project
name. It is intentionally conservative: it asks for the project name if one is missing, asks whether the GitHub
repository should be public or private, and avoids creating extra files beyond the scaffold unless the user asks for
them.

Typical prompt:

```text
Use $create-project to create a new project called my-app.
```

The skill creates this starter structure:

```text
my-app/
├── .gitignore
├── README.md
├── AGENTS.md
├── agents/
└── docs/
```

File and folder purpose:

- `.gitignore` starts the project with a place for ignored local, generated, and dependency files.
- `README.md` starts with the project name and a short placeholder description.
- `AGENTS.md` tells coding agents that agent configs belong in `agents/`, project documentation belongs in `docs/`, and
  gstack-style requests should route to matching skills before answering directly.
- `agents/` is for agent configs, task plans, subtask files, handoff notes, and related agent artifacts.
- `docs/` is for project documentation, specs, design notes, architecture notes, and decision records.

After creating the scaffold, `create-project` initializes Git, commits the initial scaffold, creates the GitHub
repository with the requested visibility, adds `origin`, and pushes the first commit.

## Using `decompose-task` and `delegate-agent-tasks` Together

These two skills are designed to work as a planning and execution pair.

Use `decompose-task` first when the work is still broad, abstract, or too large for one agent pass. It turns the goal
into agent-ready task cards and writes the plan to the current workspace's `agents/` folder, usually as
`agents/subtasks.md`. Each card should include the outcome, scope, context packet, agent instructions, acceptance
criteria, validation, dependencies, and handoff artifact.

Then use `delegate-agent-tasks` to execute those cards. It reads the task plan, decides which subtasks can run in
parallel, spawns or coordinates subagents where the host platform supports them, applies relevant validation skills,
integrates results, and commits coherent checkpoints when appropriate.

Typical flow:

```text
Use $decompose-task to break this project goal into agent-ready subtasks and write them to agents/subtasks.md.
```

Review the generated plan, then dispatch it:

```text
Use $delegate-agent-tasks to execute the task cards in agents/subtasks.md. Let implementation agents write code where the cards permit it, run appropriate validation, and commit coherent checkpoints.
```

Good handoff habits:

- Keep `decompose-task` responsible for splitting work and making context explicit.
- Keep `delegate-agent-tasks` responsible for execution, validation, integration, and commit cadence.
- If a task involves common infrastructure or reusable technology, make sure the task card includes a `deep-dive` or
  `tech-discovery` step before custom implementation.
- If a platform does not support subagents, use each task card as a manual prompt for a separate coding-agent session
  and bring the handoff artifacts back to the parent session.

To make a repo friendly to both GStack and Safe Agentic Coding, keep this structure:

```text
AGENTS.md
agents/
docs/
```

Add or keep this rule in `AGENTS.md`:

Choose the conservative privacy mode first unless you know you want broader sync:

- `off`: nothing syncs.
- `artifacts-only`: plans, designs, retros, learnings, and reviews sync.
- `full`: everything in GStack's allowlist syncs, including behavioral state.

GStack's sync is allowlist-based and scans for credential-shaped content before pushing, but you should still treat
gbrain summaries as durable memory: compact, useful, source-linked, and sanitized.

## Using `debug-runtime`

Use `debug-runtime` when a bug is hard to understand from code inspection alone. It is modeled after a Cursor-style
debug loop: form hypotheses, add temporary identifiable runtime logs, reproduce the bug, use the logs or backtraces to
pick a root cause, make the smallest fix, verify it, and remove the instrumentation before handoff.

Typical prompt:

```text
Use $debug-runtime to investigate this flaky checkout bug. Add temporary logs if needed, collect runtime evidence, fix it narrowly, and remove the instrumentation before finishing.
```

The skill is safer than guess-and-edit debugging because it requires:

- A unique `DEBUG_RUNTIME_<date>_<slug>` marker on temporary instrumentation so every debug log can be found and
  removed.
- Explicit hypotheses before adding logs.
- Sanitized logs that avoid secrets, tokens, credentials, raw user payloads, and sensitive PII.
- Runtime evidence before patching unless an existing failing test or stack trace already proves the cause.
- Verification after cleanup, not only while debug logs are still present.
- No committed instrumentation unless the user explicitly asks for durable production logging.

It pairs naturally with `write-tests` when the fix should become a regression test, and with `simplify-code` when
runtime evidence shows confusing structure caused the bug.

## Install to OpenAI Codex

Codex supports this skill folder shape directly.

Global install:

```sh
mkdir -p ~/.codex/skills
cp -R skills/* ~/.codex/skills/
```

Project-local install, when your Codex setup supports project skills:

```sh
mkdir -p .codex/skills
cp -R skills/* .codex/skills/
```

Restart Codex after installing so the skill list refreshes.

To install only one skill:

```sh
cp -R skills/simplify-code ~/.codex/skills/simplify-code
```

## Install to Claude Code

Claude Code skills use the same `SKILL.md`-based Agent Skills format.

Global install:

```sh
mkdir -p ~/.claude/skills
cp -R skills/* ~/.claude/skills/
```

Project-local install:

```sh
mkdir -p .claude/skills
cp -R skills/* .claude/skills/
```

Restart Claude Code or start a new session after installing. Invoke a skill by name, for example:

```text
Use $tech-discovery to find libraries and APIs for this app idea.
```

## Install to Claude Desktop or Claude.ai

Claude product surfaces that support custom skills may use uploaded skill packages rather than filesystem folders.

Use one skill folder at a time:

```sh
cd skills
zip -r deep-dive.zip deep-dive
```

Upload the zip through the product's Skills settings or organization skill management flow. Preserve the folder
structure so `SKILL.md` stays at the top of the skill folder inside the archive.

## Adapt for Cursor

Cursor uses project rules rather than native `SKILL.md` skill discovery.

Recommended project setup:

```sh
mkdir -p .cursor/rules
```

For each skill you want Cursor to use, create a focused `.mdc` rule. Example:

```text
.cursor/rules/simplify-code.mdc
```

Suggested content:

```markdown
---
description: Use when reviewing code complexity or simplifying implementation.
alwaysApply: false
---

Paste or summarize `skills/simplify-code/SKILL.md` here.
When needed, also paste relevant reference files from `skills/simplify-code/references/`.
```

Cursor rules work best when they are shorter than full skills. Keep the trigger description, workflow, output shape, and
any must-follow guardrails.

## Cross-Platform AGENTS.md Setup

Many tools read or at least benefit from a root `AGENTS.md`.

Recommended minimal file:

```markdown
# AGENTS

Agent configs and related files live in `agents/`.

Project documentation lives in `docs/`.

## Skill routing

When the user's request matches an available skill, route to that skill and follow its instructions before answering
directly.

Key routing rules:

- Break down broad work -> `decompose-task`
- Dispatch subagents -> `delegate-agent-tasks`
- Write or improve tests -> `write-tests`
- Expand a small prompt into an implementation brief -> `expand-task`
- Focused web research -> `deep-dive`
- Scientific or technical grounding -> `science-check`
- Runtime-first bug debugging -> `debug-runtime`
- Clean ignored files -> `update-gitignore`
- Reduce code complexity -> `simplify-code`
- Find reusable technologies for a project -> `tech-discovery`
```