# Session Skills Pack

This repository bundles the skills created or materially updated during this Codex session.

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
| `delegate-agent-task` | Coordinates subagents to execute task cards, including code-writing permissions, validation gates, and regular commit checkpoints. |
| `write-tests` | Designs and writes robust unit, integration, regression, contract, and workflow tests. |
| `expand-task` | Expands a simple prompt into a narrow, app-aware implementation brief for another agent. |
| `deep-dive` | Performs focused web research with sourced findings for startup, product, company, market, and technical questions. |
| `science-check` | Tests ideas against scientific and technical evidence, separating claims, mechanisms, evidence quality, and uncertainty. |
| `update-gitignore` | Audits a repo, updates `.gitignore`, and untracks already-tracked ignored files without deleting local copies. |
| `simplify-code` | Analyzes code complexity and finds simpler behavior-preserving implementations, including library/API alternatives via `$deep-dive`. |
| `tech-discovery` | Researches technologies, libraries, APIs, open source projects, standards, datasets, and platforms that can bootstrap a project idea. |

### Updated Supporting Skill

| Skill | Purpose |
| --- | --- |
| `create-project` | Existing project scaffold skill updated to create `agents/`, `docs/`, and an `AGENTS.md` with gstack skill-routing guidance. |

## Repository Layout

```text
session-skills/
├── README.md
└── skills/
    ├── create-project/
    ├── decompose-task/
    ├── deep-dive/
    ├── delegate-agent-task/
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
- Dispatch subagents -> `delegate-agent-task`
- Write or improve tests -> `write-tests`
- Expand a small prompt into an implementation brief -> `expand-task`
- Focused web research -> `deep-dive`
- Scientific or technical grounding -> `science-check`
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
- `delegate-agent-task` assumes the host agent has access to subagent/thread tools. If a platform does not support subagents, use its dispatch templates as manual task prompts.
- `agents/openai.yaml` is Codex UI metadata. Other platforms can ignore it.
- References under `references/` are intentionally loaded only when needed; do not inline everything into global rules unless your platform lacks file-aware context.
