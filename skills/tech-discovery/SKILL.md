---
name: tech-discovery
description: Research existing technologies, libraries, open source projects, APIs, platforms, frameworks, standards, datasets, developer tools, and implementation references that can help bootstrap or accelerate a project idea. Use when the user asks what tech can be reused for a project, how to build an idea faster with existing tools, which APIs/libraries/open source projects apply, or wants broad internet discovery across GitHub, package registries, docs, forums, papers, marketplaces, and product ecosystems.
---

# Tech Discovery

## Overview

Use this skill to turn a project idea into a practical map of reusable technologies. Search broadly across the internet, then narrow to options that actually fit the user's product goals, technical constraints, and implementation path.

This is not generic market research. The output should help an agent or engineer decide what to adopt, fork, integrate, study, or reject.

## Optional GStack and GBrain Compatibility

Use GStack and gbrain as optional memory, never as a required dependency.

Before researching, if `gbrain` is on PATH:

- Extract 2-4 concrete keywords from the project idea, capability lane, product area, or repo name.
- Run `gbrain search "<keywords>"`.
- Read at most the top 3 clearly relevant pages with `gbrain get_page "<slug>"`.
- Use memory to find prior decisions, rejected libraries, known constraints, or earlier research.
- If `gbrain` is unavailable, returns an error, or has no useful hits, continue with web and local research.

After producing a durable adoption map, save a compact summary when `gbrain` is available:

```bash
gbrain put "safe-agentic/research/<topic-slug>" --content "<markdown summary>"
```

The saved summary should include the question, recommended stack, rejected options, source links, and next spikes. Do not save secrets, credentials, raw user payloads, private keys, sensitive PII, or full copied source material. The project-local or chat artifact remains the source of truth.

## Operating Rules

- Use internet search by default. Technology availability, maintenance, docs, pricing, and API terms change often.
- Search across multiple source classes, not just blog posts or package registries.
- Prefer primary sources: official docs, GitHub repos, package registry pages, API docs, standards, papers, changelogs, pricing pages, and maintainer announcements.
- Include community evidence when useful: issue trackers, discussions, Hacker News, Reddit, Stack Overflow, Discord/forum docs, user reviews, benchmarks, and migration posts.
- Match tools to the actual project idea. Do not list popular libraries that do not reduce implementation work or risk.
- Separate adoption recommendations from exploratory leads.
- Cite sources with links and note freshness when relevant.
- Flag license, pricing, security, data/privacy, platform lock-in, and maintenance risks.

## Workflow

### Step 1: Frame the Project Idea

Extract or infer:

- Core user job and product promise.
- Key technical capabilities required.
- Preferred stack, platform, language, deployment target, and budget if known.
- Constraints: privacy, latency, scale, offline support, regulated data, open source preference, commercial use, or speed-to-prototype.
- Build stages: prototype, MVP, production hardening, growth/scale.

Ask one concise question only if the missing stack/domain would materially change the search. Otherwise state assumptions and proceed.

### Step 2: Build Capability Lanes

Break the idea into capability lanes such as:

- Frontend/UI, backend/API, auth, payments, data storage, search, AI/ML, realtime, workflows/automation, analytics, notifications, file/media handling, geospatial, integrations, deployment, observability, testing, security, compliance, datasets, and developer tooling.
- For each lane, decide whether the best reusable asset is likely a library, API, open source app, hosted platform, protocol/standard, dataset, SDK, framework feature, or reference implementation.

Read [source-lanes.md](references/source-lanes.md) when choosing where to search.

### Step 3: Search Broadly, Then Narrow

For each important capability lane:

- Search direct terms, synonyms, developer vocabulary, buyer/user vocabulary, and adjacent implementation patterns.
- Search source-specific surfaces: GitHub, package registries, official docs, API marketplaces, cloud marketplaces, papers, standards bodies, forums, and product launch archives.
- Open and inspect promising sources. Do not rely on snippets.
- Track whether each option is adoptable, forkable, study-only, or not relevant.
- Stop when more searches repeat known options or the lane has enough evidence for a decision.

Use `$deep-dive` for focused subquestions when a lane needs a deeper web pass, such as "best open source vector search options for local-first notes" or "available APIs for business entity verification in the US."

### Step 4: Score Fit

Evaluate each candidate against the project, not in isolation:

- Capability fit and missing pieces.
- Integration effort with the expected stack.
- Maturity, maintenance, docs quality, and ecosystem adoption.
- License and commercial compatibility.
- Pricing and vendor lock-in.
- Security, privacy, data retention, and compliance posture.
- Performance, reliability, scalability, and operational complexity.
- Exit strategy: replaceable API, portable data, forkability, or open standard.

Read [evaluation-rubric.md](references/evaluation-rubric.md) for scoring guidance.

### Step 5: Produce an Adoption Map

Organize results by how they help the project:

- **Adopt now**: strong fit, low integration risk, clear value.
- **Prototype/spike**: promising but needs proof.
- **Study or borrow ideas**: useful reference, not something to depend on.
- **Avoid for now**: poor fit, risky, stale, overkill, incompatible, or expensive.

Tie each recommendation to a project capability and next implementation step.

## Output Shape

Use this structure for nontrivial project ideas:

```markdown
## Project Fit Summary
[Brief framing and assumptions]

## Recommended Bootstrap Stack
| Capability | Recommendation | Why it fits | Risk/constraint | First integration step |
| --- | --- | --- | --- | --- |

## Candidate Map
### <Capability Lane>
- [Name](url) - Type: library/API/open source/platform/standard/dataset
  - Fit: Adopt now / Prototype / Study / Avoid
  - Evidence: [source-backed facts]
  - Notes: [license, pricing, maintenance, integration, alternatives]

## Open Source Projects to Inspect
| Project | What to learn or reuse | License | Health signal |
| --- | --- | --- | --- |

## APIs and Platforms
| API/platform | Use case | Pricing/limits | Lock-in or data risk |
| --- | --- | --- | --- |

## Next Spikes
- [Small experiment with acceptance criteria]
```

For small requests, use a shorter answer, but still include links and adoption judgment.

## Quality Bar

Before finalizing, check:

- The researched options cover multiple internet/source classes.
- Every recommended option maps to a project capability.
- Current source links are included.
- Open source candidates include license and health signals when available.
- APIs/platforms include pricing/limits or a note that pricing was not found.
- The answer distinguishes adopt now, prototype, study, and avoid.
- The next spikes are narrow enough for an agent to run.

## Anti-Patterns

- Do not produce a popularity list with no project fit analysis.
- Do not recommend adding dependencies without considering integration and long-term maintenance.
- Do not ignore open source alternatives when APIs/platforms exist, or vice versa.
- Do not treat stale repos as safe just because they have many stars.
- Do not hide uncertainty about licenses, pricing, security, or maintenance.
