# 6.01 — AI Governance Frameworks

## Key Insight

AI governance is not bureaucracy for its own sake — it is the mechanism that lets organisations move fast with AI without creating liabilities that surface six months later. The two most relevant frameworks (NIST AI RMF and ISO 42001) are both structured around the same insight: risk in AI systems is ongoing, not a one-time assessment. Building governance that is continuous rather than point-in-time is what separates organisations that scale AI from those that stall.

---

## Why Governance Matters Now

Three pressures are converging:

1. **Regulatory mandates are arriving.** The EU AI Act is in force. GDPR applies to AI systems that process personal data. Sector-specific regulations (financial services, healthcare, public sector) are adding AI-specific requirements.

2. **AI failures are public.** A biased hiring tool, a hallucinated legal citation, a customer-facing chatbot that says something harmful — these are reputational events, not just technical incidents.

3. **The cost of retrofitting governance is high.** Organisations that build AI features without governance frameworks find themselves unable to comply with audit requests, unable to explain system decisions, and unable to quickly disable or modify a feature that is behaving badly.

The question is not whether to govern AI — it is whether to do it proactively or reactively.

---

## NIST AI Risk Management Framework (AI RMF)

Published by the US National Institute of Standards and Technology, the AI RMF is a voluntary framework that has become a de facto standard for structured AI risk management. It is technology-neutral, applicable to any organisation, and does not require certification.

### The Four Functions

The AI RMF organises AI risk management into four functions:

| Function | What it means in practice |
|----------|--------------------------|
| **GOVERN** | Establish policies, roles, accountability, and culture. Who owns AI risk? What are the rules? |
| **MAP** | Identify and categorise AI risks for specific use cases before development begins |
| **MEASURE** | Evaluate AI systems against identified risks — during development and in production |
| **MANAGE** | Respond to risks: mitigate, accept, transfer, or avoid. Includes incident response. |

These functions are not a sequential process — they operate continuously across the lifecycle of every AI system.

### Applying the AI RMF to a Development Team

You do not need to implement the full NIST framework to get value from it. The minimum useful implementation:

**GOVERN — answer these questions:**
- Who is accountable for AI risk in your organisation? (Name, not a role title)
- What is your acceptable use policy for AI? (See template below)
- What is your incident response process for an AI feature that behaves badly?

**MAP — for each AI feature being built:**
- What is the use case? Who are the affected users?
- What could go wrong? (Classification errors, harmful outputs, PII leakage, over-reliance)
- What is the severity if it does go wrong? (Who is harmed, and how?)
- What is the likelihood? (How well-validated is the model for this task?)

**MEASURE — build these in from the start:**
- How will you know the system is working as intended? (Evaluation metrics — see Module 4)
- How will you detect when it degrades? (Monitoring — see Module 6.04)
- What is the baseline? (A golden dataset for comparison over time)

**MANAGE — have a plan before you need it:**
- Can you disable the AI feature quickly without disabling the application?
- Can you explain why the system produced a specific output?
- Do you have an audit log of AI decisions for the last 90 days?

---

## ISO 42001 — AI Management System Standard

ISO 42001 (published 2023) is the international standard for AI management systems. It follows the same high-level structure as ISO 27001 (information security) and ISO 9001 (quality management), which means organisations that already hold those certifications can extend their existing management system.

### What ISO 42001 Requires

The standard is structured around the Plan-Do-Check-Act cycle. For most organisations not pursuing formal certification, the most useful elements are:

**Policy requirements:**
- An AI policy document stating the organisation's objectives and commitments for AI use
- Clear scope: which AI systems are in scope for the management system
- Roles and responsibilities: who is accountable for AI governance

**Risk management requirements:**
- AI risk assessment process (aligned with NIST MAP)
- Documented risk treatment decisions
- Review cycle for AI risks (not a one-time assessment)

**Operational requirements:**
- Controls for human oversight where AI decisions affect individuals
- Data quality management for AI training and evaluation data
- Documentation of AI system objectives, design decisions, and testing results

**Performance evaluation:**
- Monitoring and measurement of AI system performance
- Internal audit of the AI management system
- Management review (at defined intervals)

### ISO 42001 for Development Teams

You do not need to pursue ISO 42001 certification to use the framework. The practical value is the mental model: treat AI governance as a management system with documented policies, reviewed risks, measured outcomes, and continuous improvement — not as a checklist you complete once before launch.

---

## Acceptable Use Policy — Template

Every organisation deploying AI features needs an Acceptable Use Policy. The following is a starting template — adapt it to your context.

---

**AI Acceptable Use Policy — [Organisation Name]**
*Version 1.0 — [Date]*

**Scope**: This policy applies to all AI-assisted features and tools used in [Organisation Name] products and internal systems.

**Permitted uses:**
- Code assistance, review, and documentation generation for internal development
- Customer-facing features where the AI role is disclosed to users
- Document summarisation, classification, and extraction where outputs are reviewed by a human before action is taken
- Developer productivity tooling

**Uses requiring review before implementation:**
- Any AI feature that makes decisions affecting individual customers (pricing, access, support resolution)
- Any feature processing health, financial, legal, or other sensitive personal data
- Any automated system where AI output is acted on without human review

**Prohibited uses:**
- Fully automated decision-making that denies services, employment, or access to individuals without human review
- Systems designed to deceive users about the AI nature of their interactions
- Processing of sensitive personal data without a documented legal basis
- Use of LLM providers whose data processing terms have not been reviewed against your data classification

**Accountability:**
- Product owners are responsible for completing an AI risk assessment before any in-scope feature enters development
- Engineering leads are responsible for ensuring AI features meet the technical controls defined in the AI Security Policy
- [Named role] is accountable for this policy

---

## Governance Checklist — Starting a New AI Feature

Before any AI feature enters your development backlog, answer these questions:

- [ ] **Purpose**: What problem does this solve, and why is AI the right solution?
- [ ] **Data**: What data flows to the LLM? Is any of it personal, sensitive, or regulated?
- [ ] **Output risk**: What happens when the AI is wrong? Who is affected?
- [ ] **Human oversight**: Is there a human review step before AI outputs cause real-world effects?
- [ ] **Disclosure**: Will users know they are interacting with AI?
- [ ] **Provider**: Has the LLM provider's data processing terms been reviewed for this data category?
- [ ] **Audit trail**: How will AI decisions be logged and retrievable for 90+ days?
- [ ] **Disable path**: Can this feature be turned off in under an hour if needed?

If you cannot answer all eight questions, the feature is not ready to start development.

---

## Check Your Understanding

1. A colleague argues that governance frameworks like NIST AI RMF are for large enterprises and compliance teams, not development teams. How would you respond, and what would you point to as the minimum useful governance for a team of five developers shipping AI features?

2. Your organisation wants to achieve ISO 42001 certification. You already hold ISO 27001. What work have you already done that maps to ISO 42001 requirements, and what are the AI-specific additions you need to build?

3. You are the product owner for a customer support chatbot. Use the governance checklist above to assess your feature. Which questions are hardest to answer, and why?

4. Your organisation has no AI policy today. Write the first three sentences of an acceptable use policy for a software consultancy with 200 developers. What are the highest-risk scenarios you need to address first?

5. A competitor launches an AI feature similar to yours two months before you do, without running through your governance checklist. Their feature attracts negative press coverage six months later. Retrospectively, which governance questions, if answered, would most likely have prevented the incident?
