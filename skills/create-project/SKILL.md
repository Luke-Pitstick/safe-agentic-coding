---
name: create-project
description: Create a new local project directory and GitHub repository from a user-provided project name. Use when the user asks Codex to start, scaffold, initialize, or create a new project/repo, especially when they want a matching directory, git repository, README, docs folder, agents folder, AGENTS.md, and GitHub remote.
---

# Create Project

## Optional GStack and GBrain Compatibility

When creating `AGENTS.md`, include Safe Agentic Coding and GStack routing guidance that works with or without gbrain.

If the user asks for gbrain setup while creating the project, do not invent credentials or run remote setup blindly. Give them the setup commands and ask for any required visibility, token, or remote decisions before executing.

The generated project should treat `agents/` and `docs/` as the source of truth. If gbrain is later enabled, it is optional memory/search over durable artifacts, not a replacement for project files.

## Workflow

Use the user-provided project name as both the local directory name and GitHub repository name.

If the user did not provide a project name, ask for one before taking any filesystem or GitHub action. Do not invent a name.

Before running `gh repo create`, ask whether the GitHub repository should be public or private. Treat the answer as required because visibility is not safe to assume.

DO NOT CREATE ANY FILES OR FOLDERS OTHER THAN THE ONES SPECIFIED IN THE STEPS BELOW UNLESS THE USER ASKS FOR THEM.

## Steps

1. Resolve the project name.
   - Preserve the user's exact name unless it contains path separators or shell-unsafe characters.
   - If normalization is needed, explain the normalized directory/repo name before creating anything.
   - Do not create the project inside another repository unless the user explicitly asks for that location.

2. Create and enter the new project directory.
   - Run `mkdir -p <project-name>`.
   - Use that directory as the working directory for the remaining commands.

3. Initialize Git.
   - Run `git init`.

4. Create starter project files and folders.
   - Create `.gitignore`.
   - Create `README.md` with the project name as the heading and a short placeholder description.
   - Create `docs/` for documentation.
   - Create `agents/` for AI agent configs and agent-related files.
   - Create `AGENTS.md` in the project root with this exact guidance:

```markdown
# AGENTS

Agent configs and related files live in `agents/`.

Project documentation lives in `docs/`.

## Optional memory

If GStack/gbrain is installed, agents may use it as optional memory:
- Search relevant prior context before planning: `gbrain search "<keywords>"`.
- Read only the few relevant pages needed for the task.
- Save compact summaries of durable plans, research, reviews, debug findings, and handoffs under `safe-agentic/...`.
- Keep project files in `agents/` and `docs/` as the source of truth.
- Never save secrets, credentials, raw user payloads, private keys, sensitive PII, or large code dumps to memory.
- Continue normally when `gbrain` is unavailable.

## Skill routing

When the user's request matches an available gstack skill, route to that skill and follow its instructions before answering directly. When in doubt, use the matching skill.

Key routing rules:
- Product ideas/brainstorming -> invoke /office-hours
- Strategy/scope -> invoke /plan-ceo-review
- Architecture -> invoke /plan-eng-review
- Design system/plan review -> invoke /design-consultation or /plan-design-review
- Full review pipeline -> invoke /autoplan
- Bugs/errors -> invoke /investigate
- QA/testing site behavior -> invoke /qa or /qa-only
- Code review/diff check -> invoke /review
- Visual polish -> invoke /design-review
- Ship/deploy/PR -> invoke /ship or /land-and-deploy
- Save progress -> invoke /context-save
- Resume context -> invoke /context-restore
- Author a backlog-ready spec/issue -> invoke /spec
```

5. Commit the initial scaffold.
   - Run `git add .`.
   - Run `git commit -m "Initial project scaffold"`.

6. Create the GitHub repo and push.
   - Run `gh repo create <project-name> --source . --remote origin --push --public` for public repos.
   - Run `gh repo create <project-name> --source . --remote origin --push --private` for private repos.
   - If the GitHub CLI prompts interactively, answer consistently with the user's requested repo visibility and the current local repository.

## GitHub CLI Notes

If `gh` is not authenticated, tell the user what `gh` reported and ask them to authenticate before retrying.

If the repository name already exists on GitHub or the local directory already exists, stop and ask the user whether to choose a different name or use the existing location. Do not overwrite unrelated work.

If `gh repo create --push` succeeds, report the project path and GitHub URL.
