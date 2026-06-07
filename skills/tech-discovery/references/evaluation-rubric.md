# Evaluation Rubric

Score candidates relative to the project idea, not as abstract "best tools."

## Fit Levels

- **Adopt now**: solves a core capability, integrates with expected stack, has acceptable license/pricing, and reduces implementation risk.
- **Prototype/spike**: likely useful, but needs proof around API fit, performance, pricing, data/privacy, or developer experience.
- **Study/borrow**: useful architecture, patterns, UI, tests, or algorithms, but not appropriate as a dependency.
- **Avoid for now**: stale, incompatible, risky, overkill, too expensive, poor docs, bad license fit, or weak relevance.

## Health Signals

- Recent releases or commits.
- Maintainer responsiveness in issues/discussions.
- Clear docs and examples.
- Active downstream users or ecosystem integrations.
- Security policy or advisory history.
- Stable API or documented migration path.
- Compatible license for the intended commercial/open source use.

## Risk Questions

- Does it require sending sensitive user data to a third party?
- Does pricing scale with the project's expected usage?
- Can the team replace it later without rewriting the product?
- Is the dependency larger or more complex than the problem it solves?
- Does it constrain deployment, hosting, data model, or user experience?
- Are there known incidents, deprecations, acquisition risks, or API instability?

## Recommendation Format

For each important candidate, capture:

- Capability: what project need it addresses.
- Type: library, API, open source app, platform, standard, dataset, SDK, or reference.
- Fit level: adopt/prototype/study/avoid.
- Evidence: primary source facts and community signals.
- Tradeoff: what it saves and what it costs.
- First step: smallest practical integration or inspection task.
