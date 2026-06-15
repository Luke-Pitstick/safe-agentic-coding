---
name: deep-dive
description: Perform focused internet deep dives for a user-provided question, startup idea, product validation prompt, competitor scan, market landscape check, technology comparison, or specific fact-finding task. Use when the user asks Codex to research the web, deep dive into a topic, find existing companies/products, validate an idea against the market, compare alternatives, gather sources, or answer a scoped question with current external evidence without running overly broad research.
---

# Deep Dive

## Overview

Answer a scoped research question with targeted internet searches and source-backed synthesis. This skill is intentionally narrower than open-ended research: search for the specific evidence the user asked for, do enough follow-up to avoid obvious misses, then stop with a clear answer and links.

## Optional GStack and GBrain Compatibility

Use GStack and gbrain as optional memory, never as a required dependency.

Before researching, if `gbrain` is on PATH:

- Extract 2-4 concrete keywords from the prompt, market, company, product, or technical area.
- Run `gbrain search "<keywords>"`.
- Read at most the top 3 clearly relevant pages with `gbrain get_page "<slug>"`.
- Use memory to find prior research, known constraints, previous decisions, or already-rejected options.
- If `gbrain` is unavailable, returns an error, or has no useful hits, continue with live research and local artifacts.

After producing a durable finding, save a compact summary when `gbrain` is available:

```bash
gbrain put "safe-agentic/research/<topic-slug>" --content "<markdown summary>"
```

The saved summary should include the question, answer, strongest sources, caveats, and next checks. Do not save secrets, credentials, raw user payloads, private keys, sensitive PII, or full copied source material. The project-local or chat artifact remains the source of truth.

## Operating Rules

- Use internet search by default. The skill's purpose is current external evidence.
- Keep the scope narrow. Turn broad prompts into 2-5 explicit research questions.
- Prefer primary sources: company sites, product docs, pricing pages, SEC filings, app stores, GitHub repos, papers, official datasets, standards, reputable news, or direct customer/user evidence.
- For startup validation, prioritize existing products, companies, substitutes, buyer behavior, pricing, traction signals, and unmet pain.
- Do not overfit on SEO listicles, generic market-size pages, or unsourced blog summaries.
- Search from multiple angles: direct keywords, synonyms, buyer language, competitor categories, problem phrasing, and adjacent substitutes.
- Cite sources in the final answer. Include enough links for the user to inspect.
- Clearly separate sourced facts from inference.
- If a query is high-stakes, financial, legal, medical, or safety-relevant, use official or primary sources and include uncertainty.

## Workflow

### Step 1: Frame the Research Target

Restate the user's prompt as:

- Target question
- Why it matters
- Scope boundary
- What evidence would change the answer

If the user asks something broad like "validate this startup idea," narrow it into focused questions such as:

- What existing companies or products solve this?
- How do they position and price it?
- Who appears to buy or use it?
- What substitutes do users rely on today?
- What gaps or complaints appear repeatedly?

Ask a clarifying question only when the missing detail changes the search domain materially. Otherwise make a reasonable assumption and say it.

### Step 2: Build Search Lanes

Create 2-5 search lanes before searching. Examples:

- Direct competitors: product/company names and category terms.
- Substitute workflows: how users solve the problem without a dedicated product.
- Customer evidence: reviews, forums, app stores, case studies, job posts, procurement pages.
- Pricing and business model: pricing pages, marketplace listings, public filings, partner pages.
- Technical feasibility: docs, GitHub repos, standards, papers, benchmarks.
- Timing and market change: recent news, regulations, platform shifts, funding, launches.

Read `references/query-patterns.md` when you need query examples for startup/product validation.

### Step 3: Search and Triage

Run targeted web searches for each lane:

- Start broad enough to discover vocabulary.
- Then search specific terms and source types.
- Open and inspect promising sources; do not rely only on search result snippets.
- Track source credibility and freshness.
- Prefer recent sources when the topic could have changed.
- Save only findings that directly answer the target question.

Stop searching when additional results repeat the same evidence or when the research question has enough support for a bounded conclusion.

### Step 4: Synthesize

Turn findings into a decision-useful answer:

- What exists already?
- What is meaningfully different or similar?
- What evidence supports demand or lack of demand?
- What gaps remain?
- What should the user investigate next?

For startup ideas, avoid binary "good/bad idea" conclusions unless evidence is strong. Prefer: "promising if X", "crowded unless Y", "needs validation around Z".

### Step 5: Output

Use the default final structure:

```markdown
## Research Question
[Scoped question]

## Short Answer
[3-6 sentence synthesis]

## What I Found
| Source | What it shows | Why it matters |
| --- | --- | --- |
| [Name](url) | [fact] | [implication] |

## Existing Products / Companies
- [Company/product](url): [what it does, who it serves, pricing/traction signal if found]

## Interpretation
- Sourced facts: [facts]
- Inferences: [what those facts suggest]
- Unknowns: [what still needs validation]

## Next Searches or Validation Steps
- [specific next step]
```

For smaller factual questions, use a shorter answer, but still include links.

Read `references/output-template.md` for startup validation and competitor-scan variants.

## Quality Bar

Before finalizing, check:

- The answer is scoped to the user's actual question.
- At least 3 useful sources were inspected for nontrivial research, unless the answer is intentionally tiny.
- Existing companies/products are linked when the prompt asks for startup validation.
- Facts and inferences are separated.
- Dates or freshness are mentioned when relevant.
- The final answer includes source links.
- The next steps are specific enough to execute.

## Anti-Patterns

- Do not produce broad "market research" when the user asked for a specific competitor/product/fact scan.
- Do not cite search result snippets as if they were inspected sources.
- Do not use market-size numbers without checking the source and credibility.
- Do not ignore substitutes just because they are not direct competitors.
- Do not bury uncertainty. Name what evidence is missing.
