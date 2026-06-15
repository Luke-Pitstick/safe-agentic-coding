---
name: write-docstrings
description: Add, improve, or audit verbose docstrings and equivalent API documentation comments in source code, including explicit parameter and output/return documentation for functions and methods. Use when the user asks Codex to write docstrings, document functions/classes/modules, add JSDoc/TSDoc/Rustdoc/XML/KDoc/Doxygen comments, document all files, document specified files, or make public code easier to understand without changing behavior.
---

# Write Docstrings

## Overview

Add useful, moderately verbose documentation comments to source code while preserving behavior. Treat "docstrings" broadly: use the language's idiomatic equivalent for modules, packages, classes, functions, methods, components, types, interfaces, exported constants, and public APIs.

## Optional GStack and GBrain Compatibility

Use GStack and gbrain as optional memory, never as a required dependency.

Before documenting, if `gbrain` is on PATH:

- Extract 2-4 concrete keywords from the target module, API, package, or feature area.
- Run `gbrain search "<keywords>"`.
- Read at most the top 3 clearly relevant pages with `gbrain get_page "<slug>"`.
- Use memory only for public API decisions, domain terms, prior docs guidance, or known invariants.
- If `gbrain` is unavailable, returns an error, or has no useful hits, continue from local code and docs.

After substantial documentation work, save a compact summary when `gbrain` is available:

```bash
gbrain put "safe-agentic/reviews/<docstrings-slug>" --content "<markdown summary>"
```

The saved summary should include documented APIs, conventions followed, validation run, and remaining docs gaps. Do not save secrets, credentials, raw user payloads, private keys, sensitive PII, or large code dumps. The source files remain the source of truth.

## Workflow

1. Determine scope.
   - If the user names files, only modify those files unless imports or generated documentation require a tiny adjacent change.
   - If the user says all files, document project source files and skip generated, vendored, build output, lockfiles, minified files, migrations, snapshots, and third-party code.
   - If the scope is ambiguous, inspect the repository shape and choose the likely application/library source directories.

2. Inspect existing style first.
   - Search for existing docstrings/comments in nearby files.
   - Match the repository's established tone, detail level, tags, wrapping, and formatter expectations.
   - Prefer project-specific documentation conventions over generic language defaults.

3. Select the language-native form.
   - Python: triple-quoted docstrings on modules, classes, functions, and methods.
   - JavaScript/TypeScript: JSDoc/TSDoc `/** ... */` for exported functions, classes, components, hooks, interfaces, and non-obvious module-level APIs.
   - Rust: `///` for items and `//!` for modules/crates.
   - Go: leading comments that begin with the declared identifier for exported items.
   - Java/Kotlin/Scala: Javadoc/KDoc/Scaladoc above public APIs.
   - C/C++/C#: Doxygen/XML documentation comments if the project uses them; otherwise concise leading comments for public APIs.
   - Ruby: YARD/RDoc-style comments if present; otherwise concise leading comments.
   - Shell/YAML/SQL/config: add brief header or block comments only when they clarify non-obvious intent or usage.

4. Document the contract, not the syntax.
   - Explain purpose, parameters, return values, yielded values, exceptions/errors, side effects, units, invariants, and important edge cases when they are part of the API contract.
   - For functions and methods, include explicit parameter documentation and output documentation unless the function has no inputs or no meaningful output.
   - Prefer a complete, readable description over a terse one-liner when the API has parameters, branching behavior, side effects, callbacks, generics, or error cases.
   - Mention async behavior, I/O, mutation, caching, retries, security assumptions, and concurrency constraints when relevant.
   - For UI components, document user-visible responsibility, important props, callbacks, state ownership, and accessibility assumptions.
   - For tests, document helpers, fixtures, and complex scenarios; avoid restating test names or assertions.

5. Keep comments accurate and maintainable.
   - Read enough implementation and call sites to avoid inventing behavior.
   - Do not add comments that merely paraphrase identifiers or obvious code.
   - Prefer clear multi-line docstrings over cramped summaries when detail improves comprehension.
   - Use TODO/FIXME only when the user asked for them or the repository already tracks documentation gaps that way.
   - Preserve existing public API names, behavior, imports, formatting style, and file organization.

6. Implement safely.
   - Use AST-aware or formatter-friendly edits when reasonable.
   - Avoid broad automated insertion that creates low-value boilerplate.
   - When documenting many files, work in batches by language or directory and periodically inspect diffs for repetitive or misleading comments.
   - Do not update generated documentation sites unless the user asks.

7. Validate.
   - Run the smallest relevant formatter, linter, type checker, or test command available for the touched files.
   - If validation is unavailable or too expensive, at least inspect the diff and mention the remaining risk.

## Quality Bar

Good docstrings answer questions a maintainer would actually have:

- What is this for?
- What contract does it expose?
- What inputs are accepted or rejected?
- What comes back, and in what shape or units?
- What side effects or failure modes matter?
- What context would be hard to infer from the code alone?

## Function Documentation

For every documented function or method, include these elements in the language's idiomatic format:

- Summary: one or two sentences describing what the function does and why it exists.
- Parameters: every parameter, option object field, callback, receiver-like input, or generic type parameter that affects behavior. Include expected shape, units, defaults, accepted values, mutation, ownership, and null/undefined/empty handling when relevant.
- Output: return value, yielded value, resolved promise, callback result, stream/event output, mutation-only effect, or "none" equivalent when the function intentionally returns nothing meaningful.
- Errors: exceptions, rejected promises, error returns, panics, validation failures, or important failure states when visible to callers.
- Side effects: I/O, network calls, filesystem writes, environment reads, database updates, logging, caching, metrics, state mutation, or user-visible UI changes.
- Examples: include a small example only when the repository style already uses examples or the function's usage would otherwise remain ambiguous.

Prefer the project's local convention for naming sections:

- Python: Google, NumPy, or reStructuredText style, matching nearby code. Use sections such as `Args:`, `Returns:`, `Raises:`, or their local equivalents.
- JavaScript/TypeScript: `@param`, `@returns`, `@throws`, and relevant TSDoc tags. For object parameters, document meaningful properties.
- Rust: prose plus `# Arguments`, `# Returns`, `# Errors`, `# Panics`, and `# Examples` when appropriate.
- Go: prose comments that satisfy Go style; add parameter and return details in natural language when tags are not idiomatic.
- Java/Kotlin/C#: `@param`, `@return`, `@throws`, XML tags, or KDoc/Javadoc equivalents.

When static types already show the basic type, use the docstring to explain meaning, constraints, and caller expectations rather than repeating only the type.

Avoid documentation that only says:

- The function name in prose.
- The parameter type already obvious from static types.
- Implementation steps that will drift quickly.
- Marketing language or vague adjectives.

## Scope Shortcuts

Use these defaults when the user does not specify exact files:

- Libraries: prioritize exported/public API surfaces before private helpers.
- Applications: prioritize entry points, services, controllers/routes, domain logic, hooks, shared utilities, and complex components.
- Scripts: prioritize usage, required environment variables, inputs/outputs, and destructive side effects.
- Mixed-language repos: document the language most central to the requested area first, then continue outward.

## Reporting

In the final response, summarize:

- Which files or areas were documented.
- The documentation style used.
- Any validation run.
- Any intentionally skipped areas such as generated files or ambiguous private helpers.
