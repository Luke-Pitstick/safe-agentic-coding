# Cleanup and Validation

Use this checklist before final handoff.

## Cleanup

Search for the debug marker:

```sh
rg "DEBUG_RUNTIME_"
```

Also search for common leftover patterns in touched files:

```sh
rg "console\\.debug|console\\.log|print\\(|debug-runtime|/tmp/debug-runtime|stack_info=True|new Error\\(\\)\\.stack"
```

Remove:

- Temporary log statements.
- Temporary helper functions.
- Temporary debug endpoints.
- Temporary test-only switches not meant to remain.
- Hardcoded `/tmp/debug-runtime-*` paths.
- Debug servers, viewers, or watcher scripts started for the session.

Keep only intentional durable logging that the user explicitly requested or that matches existing production logging standards.

## Validation

After cleanup, run the smallest meaningful checks:

- The original reproduction flow.
- The targeted failing test, if one exists.
- New regression or characterization tests, if added.
- Typecheck/lint/build when the touched surface is shared.

Validation must run after instrumentation removal. A fix that only works with debug code still present is not done.

## Final Evidence

Report:

- The confirmed root cause.
- The evidence that confirmed it.
- The fix and why it addresses the cause.
- The exact validation run.
- Whether instrumentation was removed.
- Any remaining risk or follow-up test gap.
