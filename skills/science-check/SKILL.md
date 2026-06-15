---
name: science-check
description: Test ideas, hypotheses, claims, designs, startup concepts, product assumptions, technical proposals, health/science thoughts, or speculative reasoning against real science, engineering constraints, evidence quality, mechanisms, and known technical facts. Use when the user wants an idea grounded in scientific literature, technical reality, physics/biology/chemistry/math/engineering principles, empirical evidence, or credible uncertainty rather than intuition.
---

# Science Check

## Overview

Stress-test an idea against scientific and technical reality. Convert the idea into concrete claims, look for mechanisms and constraints, evaluate evidence quality, and return a calibrated verdict with caveats and next tests.

Use internet search when the claim depends on current science, recent papers, products, standards, safety, medicine, law, technical specs, or any external evidence that may have changed.

## Optional GStack and GBrain Compatibility

Use GStack and gbrain as optional memory, never as a required dependency.

Before checking evidence, if `gbrain` is on PATH:

- Extract 2-4 concrete keywords from the claim, mechanism, domain, or technology.
- Run `gbrain search "<keywords>"`.
- Read at most the top 3 clearly relevant pages with `gbrain get_page "<slug>"`.
- Use memory only for prior evidence reviews, known caveats, source trails, or unresolved questions.
- If `gbrain` is unavailable, returns an error, or has no useful hits, continue with live source review.

After producing a durable evidence check, save a compact summary when `gbrain` is available:

```bash
gbrain put "safe-agentic/research/<claim-slug>" --content "<markdown summary>"
```

The saved summary should include the claim, evidence grade, strongest evidence, uncertainty, and practical implication. Do not save secrets, credentials, raw user payloads, private keys, sensitive PII, or full copied papers/articles. The project-local or chat artifact remains the source of truth.

## Operating Rules

- Break ideas into testable claims before judging them.
- Separate mechanism plausibility, empirical evidence, engineering feasibility, and product/business implications.
- Prefer primary and high-quality sources: peer-reviewed papers, systematic reviews, official technical docs, standards, textbooks, government/academic sources, reputable datasets, and direct measurements.
- Use lower-quality sources only for leads, examples, or user-reported experiences; label them accordingly.
- Look for disconfirming evidence, base rates, physical limits, biological constraints, measurement error, and simpler explanations.
- Say "unknown" when evidence is weak or absent. Do not fill gaps with confident-sounding speculation.
- Explain uncertainty plainly. Use confidence levels only when grounded in source quality and consistency.
- For medical, legal, safety, or financial topics, avoid personal advice and recommend qualified professional review when decisions could cause harm.

## Workflow

### Step 1: Parse the Idea

Restate the idea in neutral terms:

- Claim: what would need to be true.
- Domain: science/technical area involved.
- Proposed mechanism: how the idea is supposed to work.
- Practical outcome: what the user hopes follows if true.
- Stakes: what could go wrong if the idea is false.

If the idea is broad, split it into 2-5 checkable claims.

### Step 2: Choose Evidence Lanes

Search or inspect evidence across the relevant lanes:

- Mechanism: known causal pathway, physical law, biological process, algorithm, material property, or systems constraint.
- Empirical evidence: experiments, measurements, clinical trials, benchmarks, field results, replication, or observational data.
- Boundary conditions: scale, environment, population, assumptions, inputs, costs, failure modes.
- Alternatives: simpler explanations, existing solutions, conventional approaches, or known limitations.
- Technical implementation: hardware/software dependencies, standards, accuracy, latency, reliability, maintainability.

Read `references/evidence-ladder.md` when deciding how much weight to give a source.

### Step 3: Search Focused Sources

Use targeted searches, not broad deep research:

- For scientific claims, search papers, systematic reviews, textbooks, official research orgs, and reputable technical explainers.
- For engineering claims, search standards, documentation, benchmarks, source code, datasheets, and credible postmortems.
- For health/biology claims, prioritize systematic reviews, medical guidelines, government/academic sources, and human evidence when relevant.
- For startup/product claims, pair this skill with `$deep-dive` if market/company discovery is needed.

Open and inspect sources. Do not rely only on search snippets.

### Step 4: Evaluate Claims

For each claim, assign one verdict:

- Supported: good evidence and plausible mechanism.
- Plausible but unproven: mechanism fits, evidence is early/indirect.
- Mixed: credible evidence points in different directions.
- Weakly supported: evidence is low quality, anecdotal, or not directly applicable.
- Contradicted: strong evidence or constraints argue against it.
- Not enough information: the claim is underspecified or evidence is missing.

Include what evidence would change the verdict.

### Step 5: Output

Default structure:

```markdown
## Idea Tested
[Neutral restatement]

## Bottom Line
[Short calibrated verdict]

## Claim Check
| Claim | Verdict | Evidence | Confidence |
| --- | --- | --- | --- |
| [claim] | [verdict] | [source-backed summary] | [low/medium/high] |

## Mechanism and Constraints
- Mechanism: [how it could work]
- Constraints: [limits, assumptions, failure modes]
- Base rates or comparables: [if relevant]

## What Would Make This More True
- [experiment, measurement, prototype, paper, benchmark, expert review]

## What Would Falsify It
- [specific observation or test]

## Sources
- [Source](url): [why it matters]
```

For small questions, use a shorter answer but keep links and uncertainty.

Read `references/output-template.md` for fuller variants.

## Quality Bar

Before finalizing, check:

- The idea was decomposed into concrete claims.
- Each important claim has a verdict and uncertainty.
- Sources are appropriate to the domain and quality-weighted.
- Mechanism and constraints are discussed, not only citations.
- Counterevidence or failure modes were considered.
- Speculation is labeled as inference.
- The answer gives a next test or falsifier.

## Anti-Patterns

- Do not answer with "sounds plausible" without mechanisms or evidence.
- Do not treat anecdotes, press releases, and peer-reviewed evidence as equal.
- Do not cite papers by title alone; explain what they actually support.
- Do not overclaim from animal, in vitro, simulation, or toy-benchmark evidence.
- Do not ignore scale, cost, latency, reliability, population, or measurement constraints.
- Do not give medical or safety instructions as a substitute for professional guidance.
