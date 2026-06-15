# Safe Agentic Coding

Safe Agentic Coding is a small workflow kit for making coding agents more useful without letting them drift. The skills here help an agent move from idea to scoped plan, from plan to delegated execution, and from implementation to validation, review, and cleanup.

The point is not to make agents do more at random. It is to make them safer by giving them clear boundaries: decompose large work before executing it, write task cards into `agents/`, research existing tools before building custom code, validate changes with tests and reviews, commit coherent checkpoints, and keep project documentation in `docs/`.

Together, the skills support a practical agentic coding loop:

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

Each `SKILL.md` contains YAML frontmatter with `name` and `description`, followed by the instructions an agent should load when the skill matches a task.

## Included Skills

### New Skills

| Skill | Purpose |
| --- | --- |
| `decompose-task` | Splits high-level goals into agent-ready subtasks written to `agents/`, including context packets, dependencies, validation, and reuse/library checks. |
| `delegate-agent-tasks` | Coordinates subagents to execute task cards, including code-writing permissions, validation gates, and regular commit checkpoints. |
| `write-tests` | Designs and writes robust unit, integration, regression, contract, and workflow tests. |
| `expand-task` | Expands a simple prompt into a narrow, app-aware implementation brief for another agent. |
| `deep-dive` | Performs focused web research with sourced findings for startup, product, company, market, and technical questions. |
| `science-check` | Tests ideas against scientific and technical evidence, separating claims, mechanisms, evidence quality, and uncertainty. |
| `debug-runtime` | Debugs tricky bugs with temporary session-tagged instrumentation, runtime logs/backtraces, narrow fixes, verification, and cleanup. |
| `update-gitignore` | Audits a repo, updates `.gitignore`, and untracks already-tracked ignored files without deleting local copies. |
| `simplify-code` | Analyzes code complexity and finds simpler behavior-preserving implementations, including library/API alternatives via `$deep-dive`. |
| `tech-discovery` | Researches technologies, libraries, APIs, open source projects, standards, datasets, and platforms that can bootstrap a project idea. |

### Updated Supporting Skill

| Skill | Purpose |
| --- | --- |
| `create-project` | Existing project scaffold skill updated to create `agents/`, `docs/`, and an `AGENTS.md` with gstack skill-routing guidance. |

## Using `create-project`

Use `create-project` when you want an agent to start a new local project and matching GitHub repository from a project name. It is intentionally conservative: it asks for the project name if one is missing, asks whether the GitHub repository should be public or private, and avoids creating extra files beyond the scaffold unless the user asks for them.

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
- `AGENTS.md` tells coding agents that agent configs belong in `agents/`, project documentation belongs in `docs/`, and gstack-style requests should route to matching skills before answering directly.
- `agents/` is for agent configs, task plans, subtask files, handoff notes, and related agent artifacts.
- `docs/` is for project documentation, specs, design notes, architecture notes, and decision records.

After creating the scaffold, `create-project` initializes Git, commits the initial scaffold, creates the GitHub repository with the requested visibility, adds `origin`, and pushes the first commit.

## Using `decompose-task` and `delegate-agent-tasks` Together

These two skills are designed to work as a planning and execution pair.

Use `decompose-task` first when the work is still broad, abstract, or too large for one agent pass. It turns the goal into agent-ready task cards and writes the plan to the current workspace's `agents/` folder, usually as `agents/subtasks.md`. Each card should include the outcome, scope, context packet, agent instructions, acceptance criteria, validation, dependencies, and handoff artifact.

Then use `delegate-agent-tasks` to execute those cards. It reads the task plan, decides which subtasks can run in parallel, spawns or coordinates subagents where the host platform supports them, applies relevant validation skills, integrates results, and commits coherent checkpoints when appropriate.

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
- If a task involves common infrastructure or reusable technology, make sure the task card includes a `deep-dive` or `tech-discovery` step before custom implementation.
- If a platform does not support subagents, use each task card as a manual prompt for a separate coding-agent session and bring the handoff artifacts back to the parent session.

## GStack and GBrain Compatibility

These skills are now designed to cooperate with GStack and gbrain when they are available, while still working from plain project files when they are not.

The source of truth stays simple:

```text
agents/   -> task cards, briefs, handoffs, reviews, debug notes
docs/     -> specs, architecture notes, research, decisions
gbrain    -> optional memory/search over compact summaries
```

When `gbrain` is on PATH, the skills use it in a lightweight way:

- Before work, extract 2-4 concrete keywords and run `gbrain search "<keywords>"`.
- Read only the top few relevant pages with `gbrain get_page "<slug>"`.
- Use retrieved memory to avoid repeated work, honor prior decisions, and surface known risks.
- After durable work, save a compact summary under `safe-agentic/...` with `gbrain put`.
- Continue normally when gbrain is missing, empty, or temporarily unavailable.

The skills do not save secrets, credentials, raw user payloads, private keys, sensitive PII, large code dumps, or full logs to gbrain. Project files and Git history remain authoritative.

The default namespaces are:

```text
safe-agentic/tasks/<slug>
safe-agentic/briefs/<slug>
safe-agentic/research/<slug>
safe-agentic/reviews/<slug>
safe-agentic/debug/<slug>
safe-agentic/handoffs/<slug>
```

## Set Up GBrain for Any Repo

From a repo where GStack is installed or available to Codex, use the GStack setup skill:

```text
Use $setup-gbrain to set up gbrain for this repo.
```

That setup flow is responsible for installing or verifying the `gbrain` CLI, initializing a local brain such as PGLite or a supported remote backend, registering the MCP integration when needed, and setting the trust policy for the endpoint.

After setup, verify from the repo:

```sh
gbrain doctor --fast --json
gbrain search "<repo or feature keyword>"
```

Then save and retrieve a tiny test page:

```sh
cat > /tmp/gbrain-setup-check.md <<'EOF'
---
title: Setup Check
tags: [safe-agentic, setup-check]
---
GBrain can save and retrieve Safe Agentic Coding summaries for this repo.
EOF

gbrain put "safe-agentic/test/setup-check" --content "$(cat /tmp/gbrain-setup-check.md)"

gbrain get "safe-agentic/test/setup-check"
gbrain search "setup-check"
```

To make a repo friendly to both GStack and Safe Agentic Coding, keep this structure:

```text
AGENTS.md
agents/
docs/
```

Add or keep this rule in `AGENTS.md`:

```markdown
## Optional memory

If GStack/gbrain is installed, agents may use it as optional memory:
- Search relevant prior context before planning: `gbrain search "<keywords>"`.
- Read only the few relevant pages needed for the task.
- Save compact summaries of durable plans, research, reviews, debug findings, and handoffs under `safe-agentic/...`.
- Keep project files in `agents/` and `docs/` as the source of truth.
- Never save secrets, credentials, raw user payloads, private keys, sensitive PII, or large code dumps to memory.
- Continue normally when `gbrain` is unavailable.
```

For cross-machine artifacts memory, use GStack's artifacts sync after local gbrain works. Newer GStack versions use `gstack-artifacts-init`; older docs may call the same idea `gstack-brain-init`.

```sh
gstack-artifacts-init
gstack-brain-sync --status
```

For repo code indexing, run the GStack sync skill from the repo:

```text
Use $sync-gbrain to sync this repo with gbrain.
```

Choose the conservative privacy mode first unless you know you want broader sync:

- `off`: nothing syncs.
- `artifacts-only`: plans, designs, retros, learnings, and reviews sync.
- `full`: everything in GStack's allowlist syncs, including behavioral state.

GStack's sync is allowlist-based and scans for credential-shaped content before pushing, but you should still treat gbrain summaries as durable memory: compact, useful, source-linked, and sanitized.

## Using `debug-runtime`

Use `debug-runtime` when a bug is hard to understand from code inspection alone. It is modeled after a Cursor-style debug loop: form hypotheses, add temporary identifiable runtime logs, reproduce the bug, use the logs or backtraces to pick a root cause, make the smallest fix, verify it, and remove the instrumentation before handoff.

Typical prompt:

```text
Use $debug-runtime to investigate this flaky checkout bug. Add temporary logs if needed, collect runtime evidence, fix it narrowly, and remove the instrumentation before finishing.
```

The skill is safer than guess-and-edit debugging because it requires:

- A unique `DEBUG_RUNTIME_<date>_<slug>` marker on temporary instrumentation so every debug log can be found and removed.
- Explicit hypotheses before adding logs.
- Sanitized logs that avoid secrets, tokens, credentials, raw user payloads, and sensitive PII.
- Runtime evidence before patching unless an existing failing test or stack trace already proves the cause.
- Verification after cleanup, not only while debug logs are still present.
- No committed instrumentation unless the user explicitly asks for durable production logging.

It pairs naturally with `write-tests` when the fix should become a regression test, and with `simplify-code` when runtime evidence shows confusing structure caused the bug.

## Repository Layout

```text
session-skills/
├── README.md
└── skills/
    ├── create-project/
    ├── debug-runtime/
    ├── decompose-task/
    ├── deep-dive/
    ├── delegate-agent-tasks/
    ├── expand-task/
    ├── science-check/
    ├── simplify-code/
    ├── tech-discovery/
    ├── update-gitignore/
    └── write-tests/
```

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

Upload the zip through the product's Skills settings or organization skill management flow. Preserve the folder structure so `SKILL.md` stays at the top of the skill folder inside the archive.

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

Cursor rules work best when they are shorter than full skills. Keep the trigger description, workflow, output shape, and any must-follow guardrails.

## Adapt for Windsurf

Windsurf uses rules and memories rather than the Codex/Claude skill folder format.

Use one of the rule mechanisms supported by your Windsurf version, such as the project rules UI or rule files. A practical adaptation is:

```text
.windsurfrules
```

or, if your version supports a rules directory:

```text
.windsurf/rules/<skill-name>.md
```

For each skill, paste a concise version of `SKILL.md`:

- Keep the `description` as the trigger.
- Keep the workflow and guardrails.
- Link or paste only the reference files that matter for the project.

## Adapt for Cline

Cline supports rules in `.clinerules/` and also detects several other instruction formats, including `AGENTS.md`.

Project rules:

```sh
mkdir -p .clinerules
```

Then create files such as:

```text
.clinerules/01-decompose-task.md
.clinerules/02-write-tests.md
.clinerules/03-simplify-code.md
```

Use the corresponding `SKILL.md` body as the rule content. Keep each rule focused so Cline can apply it predictably.

Global rules on macOS/Linux commonly live under:

```text
~/Documents/Cline/Rules
```

## Adapt for Roo Code

Roo Code commonly uses project rules rather than native skill folders.

Create project rules:

```sh
mkdir -p .roo/rules
```

Then convert skills into rule files:

```text
.roo/rules/simplify-code.md
.roo/rules/tech-discovery.md
```

Paste the relevant `SKILL.md` content and keep references nearby, either inline or in a docs folder that the rule points to.

## Adapt for GitHub Copilot Coding Agent

GitHub Copilot coding agent and Copilot Chat are typically guided through repository instructions rather than skills.

Recommended setup:

```text
.github/copilot-instructions.md
```

For broad routing, summarize the skills in that file. For longer workflows, store the full skills under:

```text
docs/agent-skills/<skill-name>.md
```

Then point Copilot instructions to the relevant docs:

```markdown
When asked to simplify code, follow `docs/agent-skills/simplify-code.md`.
When asked to discover libraries/APIs for a project, follow `docs/agent-skills/tech-discovery.md`.
```

## Cross-Platform AGENTS.md Setup

Many tools read or at least benefit from a root `AGENTS.md`.

Recommended minimal file:

```markdown
# AGENTS

Agent configs and related files live in `agents/`.

Project documentation lives in `docs/`.

## Skill routing

When the user's request matches an available skill, route to that skill and follow its instructions before answering directly.

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

## Validation

For Codex-compatible skills, validate each folder with the Codex skill validator when available:

```sh
uv run --with pyyaml python ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py skills/deep-dive
```

Repeat for each skill you modify.

## Notes

- Skills that mention `$deep-dive` assume the `deep-dive` skill is installed alongside them.
- `delegate-agent-tasks` assumes the host agent has access to subagent/thread tools. If a platform does not support subagents, use its dispatch templates as manual task prompts.
- `agents/openai.yaml` is Codex UI metadata. Other platforms can ignore it.
- References under `references/` are intentionally loaded only when needed; do not inline everything into global rules unless your platform lacks file-aware context.
