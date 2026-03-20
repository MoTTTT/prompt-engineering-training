# 6.05 — Change Management and ROI

## Key Insight

The most common reason AI adoption stalls in organisations is not the technology — it is the people and process around it. Developers ship a working AI feature and then nothing happens, because the workflow it was supposed to improve was not changed, because the people it was supposed to help do not trust it, or because nobody measured whether it was working. Getting AI into productive use requires deliberate change management alongside the technical work.

---

## Adoption Patterns

AI adoption in development organisations follows two patterns. Both work. Both have failure modes.

### Pattern 1 — Grassroots (bottom-up)

Individual developers start using AI tools independently. Usage spreads through informal recommendation. The organisation discovers it has significant AI usage before it has any policy.

**Where this succeeds:** High developer enthusiasm, fast iteration, real use case validation before investment.

**Where this fails:** Tool sprawl (10 different AI products, no consolidated contracts or visibility), governance gaps (no DPAs, no acceptable use policy, data flowing to unapproved providers), inability to scale (each developer doing their own thing, no shared patterns or institutional learning).

### Pattern 2 — Platform mandate (top-down)

Leadership decides AI adoption is a strategic priority. A platform is selected and mandated. Training is rolled out.

**Where this succeeds:** Consistent tooling, governance from the start, institutional knowledge captured centrally.

**Where this fails:** The platform does not match what developers actually want to build. Training is theoretical, not practical. No autonomy → low engagement → developers use approved tools to check a box and use other tools actually.

### The Hybrid That Works

The organisations that adopt AI most successfully do both: platform team defines the guardrails (acceptable use policy, approved providers, shared patterns), but developers have genuine autonomy within those guardrails. Training is practical (build something) rather than theoretical (read the slides).

The platform team's job is to make the right way the easy way — not to prevent people from doing the wrong thing, but to ensure the right thing is accessible enough that developers choose it.

---

## Measuring AI Value

"It feels faster" is not a business case. "It saved 2.5 hours per developer per week" is. The difference is measurement.

### What to Measure

**Developer productivity proxies:**
- Time from ticket to PR (average cycle time)
- Defect escape rate (bugs found in QA vs bugs found in production)
- Code review round-trips (number of review cycles per PR)
- Time spent on specific task types (documentation, test writing, boilerplate)

The right approach: measure baseline before AI adoption, measure the same metrics after, control for other variables.

**Feature quality proxies:**
- Customer support ticket rate (for AI-assisted features)
- Mean time to resolve (for AI-assisted support tools)
- User task completion rate (for user-facing AI features)

**Cost proxies:**
- Developer hours on specific task categories (before and after AI assistance)
- Time from bug report to fix (for AI-assisted debugging)
- Documentation completeness score (before and after AI writing assistance)

### Building the Business Case

A simple structure that works with most stakeholders:

```
The Problem
[State the specific inefficiency in quantified terms]
"Our developers spend an average of 4 hours per week writing boilerplate
configuration and documentation. At £600/day fully loaded cost, this is
£600/developer/month."

The Proposal
[State what you are proposing to do, specifically]
"Deploy an AI coding assistant to the 20-person development team, covering
code completion, documentation generation, and test scaffolding."

The Investment
[State the full cost — licence, implementation, training time]
"Software: £X/month. Implementation and configuration: 5 days. Team
training: 1 day per developer = 20 days."

The Expected Return
[State conservative numbers with assumptions visible]
"Target: 20% reduction in boilerplate and documentation time.
Conservative assumption: 1 hour/week/developer recovered.
Value: 20 developers × 1 hour × £75/hour = £1,500/month.
Payback period at this rate: [X] months."

The Measurement Plan
[State specifically how you will know if it is working]
"We will measure ticket-to-PR cycle time and developer-reported time on
documentation tasks, comparing a 3-month baseline against 3 months
post-deployment."
```

The two most important elements of this structure: visible assumptions (so stakeholders can challenge them) and a measurement plan (so you can report back).

### When the Numbers Do Not Add Up Directly

Some AI value is not easily monetised:
- Developer experience and retention ("I would leave if they took it away")
- Reduced cognitive load and error rate on tedious tasks
- Faster onboarding for new team members

These are real. But they are harder to quantify and harder to sell to a budget holder. Lead with the quantifiable numbers, and treat the softer benefits as supporting evidence, not the primary case.

---

## Common Resistance Patterns

### "This will replace developers"

The most common fear. It is real, even if the specific fear is often unfounded.

**How to address it:** Be honest about what AI is actually good at (the tedious, repetitive, boilerplate parts of the job) and what it remains poor at (architectural decisions, nuanced debugging, understanding business context). Show examples of where AI assistance frees developers to do more interesting work, not less work. Do not dismiss the concern — acknowledge it and address it directly.

### "I don't trust the output"

This is a healthy instinct, not a problem to overcome. The correct response is not to argue that the AI is trustworthy — it is to design workflows where human review catches AI errors before they cause problems.

**How to address it:** Build in review steps. Start with low-stakes tasks where mistakes are cheap. Build team norms around treating AI output as a first draft, not a final answer.

### "It's not in my workflow"

AI tools that require context-switching do not get used. Tools that are in the existing workflow do.

**How to address it:** Prioritise integrations that are in the developer's existing tools (IDE plugins, PR review integrations, ticketing system plugins) over standalone tools that require separate logins and separate context.

### "The governance overhead isn't worth it"

This is the concern that doing AI properly (DPAs, risk assessments, acceptable use policies) is more overhead than the benefit justifies.

**How to address it:** The overhead of governance is front-loaded. Once the DPA is in place, once the acceptable use policy exists, the per-feature governance overhead is small. The cost of not having governance surfaces retrospectively — when a regulator asks, when an incident occurs, when a customer complains.

---

## Explaining AI Trade-Offs to Non-Technical Stakeholders

Three framing tools that work:

**The "first draft" framing:** AI produces first drafts. Humans review, correct, and finalise. This is how AI-assisted work should be designed — not autonomous, but accelerated. Most stakeholders are comfortable with this model because it matches how they already use tools (spell-checkers, autocomplete).

**The "probability, not certainty" framing:** AI output is probabilistic. It is right most of the time on well-defined tasks, but never guaranteed. Design for the error cases, not just the happy path. Stakeholders who understand this stop expecting AI to be perfect and start asking the right question: is the error rate acceptable given the use case?

**The "leverage, not replacement" framing:** AI increases leverage for existing skills. A junior developer with AI assistance can produce more, but the quality ceiling is still set by the skills they bring to the review process. This is why AI and skill development are complements, not substitutes.

---

## What "Production-Ready AI" Means Organisationally

A feature that passes technical review is not necessarily production-ready. Production-ready AI requires:

**Technically:**
- [ ] Evaluation against a golden dataset with documented quality bar
- [ ] Structured logging with audit trail
- [ ] Monitoring and alerting for quality degradation
- [ ] Cost controls (max_tokens, rate limiting, budget alerts)
- [ ] Model version pinned
- [ ] Graceful degradation if the AI feature is unavailable

**Organisationally:**
- [ ] Risk classification documented and reviewed
- [ ] Acceptable use case confirmed against policy
- [ ] Data processing reviewed (DPA in place if personal data)
- [ ] Disclosure implemented if user-facing
- [ ] Human oversight mechanism defined (where required)
- [ ] Incident response process covers this feature
- [ ] Team trained on what to do if the feature behaves badly

Both lists need to be satisfied before you ship to production. The technical checklist is easier to tick — most of the governance work is in the organisational checklist.

---

## Check Your Understanding

1. Your company is a 50-person software consultancy. You want to adopt AI coding assistants across the team. Design a rollout plan that addresses both the grassroots and platform mandate concerns. Who do you involve, in what order, and what do you measure?

2. You need to present a business case for AI-assisted code review to a CTO who is sceptical of "AI hype." Write the one-page version of the business case using the structure in this section. Make reasonable assumptions and state them explicitly.

3. A senior developer says "I tried GitHub Copilot, it was useless for my kind of work, AI coding tools are a gimmick." How do you respond in a way that takes their experience seriously while still making the case for a structured evaluation?

4. Six months after AI tool adoption, developer usage has dropped from 80% to 40% of the team. How would you diagnose the cause, and what interventions would you consider?

5. Your compliance team says all AI features need a risk assessment before development starts, and the risk assessment template takes 4 hours to complete. Developers are skipping it. What would you change about the process to get compliance without the bottleneck?
