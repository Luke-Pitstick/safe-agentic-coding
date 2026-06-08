---
name: debug-runtime
description: Debug tricky, hard-to-reproduce, or unclear bugs by adding temporary identifiable runtime instrumentation, collecting logs/backtraces/runtime evidence, narrowing hypotheses, applying a targeted fix, verifying with tests or user reproduction, and removing instrumentation before final handoff. Use when the user asks to debug a bug with logs, runtime traces, backtraces, reproduction evidence, or a Cursor-style debug loop rather than speculative code inspection.
---

# Debug Runtime

## Overview

Use this skill for bugs where static inspection is not enough. The workflow is evidence-first: form hypotheses, add minimal temporary instrumentation, reproduce the bug, use runtime facts to choose the fix, verify, and remove the instrumentation.

Pair with `$write-tests` when a regression test or characterization test can lock in the fix. Pair with `$simplify-code` only after the root cause is understood and the bug is caused by confusing structure.

## Operating Rules

- Prefer runtime evidence over model guesses.
- Keep instrumentation temporary, searchable, and tied to one debug session ID.
- Do not log secrets, tokens, credentials, full user payloads, private keys, auth headers, sensitive PII, or regulated data.
- Do not commit instrumentation unless the user explicitly asks for durable logging.
- Preserve user changes and keep the final diff minimal.
- Do not leave local servers, file watchers, or background processes running.
- Ask one concise question only when reproduction is impossible without it.

## Workflow

### Step 1: Frame the Bug

Capture:

- Observed behavior.
- Expected behavior.
- Reproduction steps or the smallest command/test/page flow likely to reproduce it.
- Environment: OS, runtime, browser, server, flags, seed data, feature flags, versions, and deployment target when relevant.
- Affected files, recent changes, stack traces, logs, screenshots, or failing tests.

If reproduction is unknown, identify the most likely reproduction path and state assumptions.

### Step 2: Generate Hypotheses

Produce 3-5 plausible causes. For each hypothesis, name:

- What runtime evidence would confirm it.
- What runtime evidence would disprove it.
- Where to instrument to observe that evidence.

Do not patch before the evidence points to a cause unless the bug is already fully explained by an existing failing test or stack trace.

### Step 3: Add Temporary Instrumentation

Generate a unique session ID such as:

```text
DEBUG_RUNTIME_<YYYYMMDD>_<short-random-or-task-slug>
```

Insert minimal logs near:

- Decision points.
- Data transformations.
- Async boundaries and race-prone callbacks.
- State changes.
- API/database/cache calls.
- Error handlers.
- UI event handlers and render boundaries.

Include file, function, event, timestamp, relevant sanitized values, and stack/backtrace when useful.

Read [instrumentation-patterns.md](references/instrumentation-patterns.md) for language and environment patterns.

### Step 4: Collect Runtime Evidence

Ask the user to reproduce the bug, or run the smallest local command/test that reproduces it.

Prefer structured JSONL logs when feasible. Use:

- Browser console logs or an optional local log endpoint for browser bugs.
- Existing app logger, stdout/stderr, test output, or `/tmp/debug-runtime-<id>.log` for backend/CLI bugs.
- Native stack/backtrace facilities when the runtime supports them.

Capture the command or reproduction flow used and the relevant log excerpt. Avoid pasting huge logs into the final answer; summarize and cite the key lines.

### Step 5: Analyze and Iterate

Map each log/backtrace to the hypotheses:

- Confirmed.
- Disproved.
- Still unknown.

If evidence is insufficient, add narrower instrumentation and reproduce again. Remove or avoid broad noisy logs that do not answer a hypothesis.

### Step 6: Patch Narrowly

Make the smallest behavior-preserving fix that addresses the observed cause.

Avoid broad refactors during debugging unless:

- The runtime evidence shows the structure caused the bug.
- The refactor is smaller and safer than a local patch.
- Tests or reproduction steps can verify the behavior.

### Step 7: Verify

Run the reproduction, targeted tests, or user verification flow again.

If fixed:

- Remove all temporary instrumentation tied to the session ID.
- Stop any debug servers/watchers.
- Re-run the relevant validation after cleanup.

If not fixed:

- Keep the evidence.
- Revise hypotheses.
- Add narrower instrumentation and loop.

Read [cleanup-and-validation.md](references/cleanup-and-validation.md) for the final checklist.

### Step 8: Report

Use this final shape:

```markdown
## Debug Summary
- Bug:
- Reproduction:
- Root cause:
- Evidence:
- Fix:
- Validation:
- Instrumentation removed: yes/no
- Remaining risks:
```

## Quality Bar

Before finalizing, check:

- The root cause is supported by runtime evidence, a failing test, or a stack trace.
- Temporary instrumentation is removed unless explicitly requested.
- No sensitive data was logged.
- Any background process started for debugging has been stopped.
- The final diff is scoped to the fix and any intentional tests.
- Validation was run after cleanup, not only while debug logs were present.

## Anti-Patterns

- Do not add logs everywhere without hypotheses.
- Do not fix based only on the first plausible log line if other evidence contradicts it.
- Do not leave `console.log`, print statements, debug endpoints, or `/tmp` file paths in production code.
- Do not make a sweeping rewrite to hide uncertainty.
- Do not ask the user to reproduce repeatedly without explaining what new evidence each run should collect.
