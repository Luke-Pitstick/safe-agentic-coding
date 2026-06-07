---
name: update-gitignore
description: Audit a Git repository's current files, generated artifacts, local config, caches, secrets, build outputs, and tracked ignored files; update .gitignore intelligently; and untrack files that should be ignored without deleting local working copies. Use when the user asks to update .gitignore, clean ignored files from the repo, stop tracking generated/local files, or check whether ignored files have already been committed or pushed.
---

# Update Gitignore

## Overview

Use this skill to make `.gitignore` match the actual repo: inspect the working tree, identify files that should not be versioned, add safe ignore patterns, then remove already-tracked ignored files from Git with `git rm --cached` while preserving local copies.

## Safety Rules

- Work only inside a Git repo. Start with `git rev-parse --show-toplevel` and operate from that root.
- Never delete the user's local files as part of normal cleanup. Use `git rm --cached -- <path>` to remove files from the index while leaving files on disk.
- Do not ignore source code, lockfiles, migrations, committed config, docs, or assets unless they are clearly generated/local or the user confirms the tradeoff.
- Treat secrets specially. If a credential, private key, token dump, or `.env` containing real secrets is already tracked or committed, update ignore rules and untrack it, but also warn that credential rotation and history cleanup may be required. Do not rewrite history unless explicitly asked.
- Stage only files you intentionally changed, and do not disturb unrelated user edits.

## Workflow

1. **Locate repo and read existing rules**
   - Run `git rev-parse --show-toplevel`.
   - Read `.gitignore` if present, plus `.git/info/exclude` only for context.
   - Check `git config --get core.excludesFile` and read the global ignore file if it exists and is accessible.

2. **Inventory files**
   - Use `git status --short --ignored`, `git ls-files`, and `git ls-files --others --exclude-standard`.
   - Use `rg --files -uu` or `find` only when needed to see ignored/untracked generated directories, avoiding huge binary dumps.
   - Identify ecosystems from manifests such as `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`, `build.gradle`, `Gemfile`, `composer.json`, `.xcodeproj`, `.xcworkspace`, `Package.swift`, `*.csproj`, and Docker or deployment files.

3. **Choose ignore candidates**
   - Prefer narrow, conventional patterns for generated artifacts, dependency directories, caches, local env files, logs, coverage, editor noise, temp files, and OS metadata.
   - Load [ignore-patterns.md](references/ignore-patterns.md) when choosing ecosystem-specific patterns or deciding whether a candidate should be kept.
   - Keep examples/templates such as `.env.example`, `.env.sample`, and documented fixture files tracked.

4. **Update `.gitignore`**
   - Create `.gitignore` if missing.
   - Preserve existing content and comments.
   - Add grouped rules with short comments only where helpful.
   - Avoid duplicate patterns; normalize exact duplicates before appending.
   - Prefer root-anchored patterns such as `/dist/` when the artifact is repo-root-specific, and unanchored patterns such as `*.log` only when appropriate globally.

5. **Verify ignored behavior**
   - Use `git check-ignore -v -- <path>` for representative candidates.
   - Run `git ls-files -ci --exclude-standard` after editing to find tracked files now matched by ignore rules.

6. **Untrack already-tracked ignored files**
   - For each path from `git ls-files -ci --exclude-standard`, decide whether it truly should leave the repo.
   - Use `git rm --cached -- <file>` or `git rm -r --cached -- <dir>` to remove it from Git without deleting the working copy.
   - If a path is sensitive, check whether it appears in commits or upstream before finalizing the report.
   - Load [git-commands.md](references/git-commands.md) for exact command patterns.

7. **Check committed/pushed status**
   - Use `git log --oneline -- <path>` to see whether the file has been committed before.
   - If an upstream branch exists, use `git log @{u} --oneline -- <path>` to see whether the upstream history contains it.
   - Report committed or pushed sensitive files clearly and recommend rotation/history cleanup as a separate explicit task.

8. **Validate and report**
   - Show the `.gitignore` diff and any `git rm --cached` staged removals.
   - Run `git status --short --ignored`.
   - Run `git ls-files -ci --exclude-standard`; the ideal result is empty, unless intentionally tracked exceptions remain.
   - Final response should list added patterns, untracked paths, any tracked ignored files left alone, and any pushed/secret risk.

## Output Shape

When completing the task, report:

- `.gitignore` patterns added or changed.
- Files removed from Git tracking, explicitly noting local files remain on disk.
- Whether any ignored files had already been committed or appear in upstream history.
- Validation commands run and the result.
- Any follow-up needed for secret rotation, history cleanup, or ambiguous candidates.
