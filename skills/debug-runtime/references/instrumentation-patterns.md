# Instrumentation Patterns

Use these patterns as guidance. Match the repo's logger and conventions when they exist.

## Session ID

Every temporary log should include a unique marker:

```text
DEBUG_RUNTIME_<YYYYMMDD>_<slug>
```

Use this marker to find and remove instrumentation:

```sh
rg "DEBUG_RUNTIME_"
```

## Structured Log Shape

Prefer one event per line:

```json
{"debug_id":"DEBUG_RUNTIME_20260608_login","event":"auth_decision","file":"auth.ts","fn":"canLogin","user_id_hash":"...","state":"denied","reason":"missing_session","ts":"2026-06-08T12:00:00Z"}
```

Include:

- `debug_id`
- `event`
- file/function/component
- sanitized values
- branch or decision taken
- timestamp
- stack/backtrace only when useful

## JavaScript and TypeScript

Browser or Node:

```js
console.debug("DEBUG_RUNTIME_20260608_case", {
  event: "state_transition",
  file: "src/example.ts",
  fn: "handleSubmit",
  before,
  after,
  stack: new Error().stack,
});
```

Prefer existing loggers when the app has one. Do not log secrets, auth headers, raw request bodies, or full user records.

## Python

```python
logger.warning(
    "DEBUG_RUNTIME_20260608_case event=%s state=%s",
    "branch_taken",
    safe_state,
    stack_info=True,
)
```

For isolated CLI/test debugging, writing JSONL to `/tmp/debug-runtime-<id>.log` is acceptable if the file path does not enter committed code.

## Backend Services

- Log request IDs, job IDs, sanitized entity IDs, state names, branch decisions, and error classes.
- Avoid payloads; log shape, counts, hashes, or redacted summaries.
- Prefer existing correlation IDs.
- If adding a temporary endpoint or log collector, remove it before final handoff.

## UI Bugs

- Log user actions, component state transitions, props that affect rendering, layout branch choices, and async completion order.
- For visual bugs, pair logs with screenshots or exact reproduction steps.
- Avoid logging full form contents.

## Concurrency and Timing Bugs

- Log start/end times, promise/job IDs, cancellation/abort signals, lock acquisition/release, queue length, and ordering.
- Keep logs narrow enough to compare two reproduction runs.
