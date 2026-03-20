# 6.03 — Regulatory Landscape

## Key Insight

Regulation of AI is not theoretical — the EU AI Act is in force, GDPR applies to most AI systems that process personal data, and sector-specific requirements (financial services, healthcare, public sector) are adding AI-specific obligations on top. The practical implication for development teams is not that you need to become compliance experts, but that you need to ask three questions for every AI feature you build: what risk tier does this fall into, what data does it process, and what disclosure obligations apply?

---

## The EU AI Act

The EU AI Act is the most comprehensive AI regulation to date. It applies to any AI system deployed in the EU — including systems deployed by non-EU companies to EU users.

### Risk Tiers

The Act classifies AI systems into four risk tiers:

| Tier | Examples | Obligations |
|------|----------|-------------|
| **Unacceptable risk** | Social scoring, real-time biometric surveillance, subliminal manipulation | **Prohibited** — cannot be deployed |
| **High risk** | Recruitment tools, credit scoring, medical diagnostics, educational assessment, law enforcement | Full compliance: conformity assessment, registration, human oversight, documentation |
| **Limited risk** | Customer-facing chatbots, AI-generated content | **Disclosure required**: users must be told they are interacting with AI |
| **Minimal risk** | Spam filters, recommendation engines, AI-assisted content tools | Voluntary codes of practice; no mandatory requirements |

### What This Means for Development Teams

**Most developer-built AI features fall into "limited risk."** A customer support chatbot, a documentation assistant, a code reviewer — these are limited-risk applications. The primary obligation is **disclosure**: users must know they are interacting with an AI system.

**The high-risk tier is where you must pay attention.** If your feature is involved in hiring decisions, loan approvals, benefit determinations, or medical recommendations — you are in the high-risk tier. This requires:
- A conformity assessment before deployment
- Documented risk management system
- Human oversight mechanisms
- Technical documentation (including training data description, intended purpose, and known limitations)
- Registration in the EU AI Act database

**General-purpose AI models (GPAIs)** — models like Claude, GPT-4, and Gemini — have their own transparency obligations under the Act. As an application developer using a GPAI via API, you inherit some of these obligations and need to understand what your provider publishes about their model.

### Practical Disclosure Implementation

The minimum disclosure for a limited-risk AI chatbot:

```
Option 1 — UI label: "AI-powered assistant"
Option 2 — First message: "Hi, I'm an AI assistant. I can help you with..."
Option 3 — Terms of service disclosure (weakest — least visible to users)
```

Best practice: label in the UI AND identify as AI in the first message.

---

## GDPR and AI

GDPR predates the EU AI Act but applies to any AI system that processes personal data about EU individuals. The intersection points:

### Data Minimisation

GDPR requires you to collect only the personal data necessary for a specified purpose. For AI systems, this means:
- Do not send more personal data to the LLM than is required for the specific task
- If you can complete the task with anonymised or pseudonymised data, use that
- Consider whether RAG corpus documents need personal data, or whether it can be removed at ingestion time

### Lawful Basis

You need a lawful basis for processing personal data in an AI system. For most enterprise applications:
- **Legitimate interests** covers many internal productivity tools (subject to balancing test)
- **Contract performance** covers customer-facing features where AI is part of the service delivery
- **Consent** is required where users have a reasonable expectation that their data is not being used in this way

### Right to Explanation

Article 22 GDPR gives individuals the right not to be subject to purely automated decision-making that produces significant effects. If your AI system makes or strongly influences decisions affecting individuals — accept/reject, price, access — you need a human review step OR an appropriate exemption.

### Data Processor vs Data Controller

When you send data to an LLM API provider:
- You are the **data controller** (you decide what to process and why)
- The API provider is the **data processor** (they process on your behalf)

This means you need a Data Processing Agreement (DPA) with the provider. Most major providers offer standard DPAs. Check that your DPA covers the data categories you send and the data retention periods you require.

---

## Sector-Specific Considerations

### Financial Services (FCA / PRA in UK, EBA in EU)

Regulators in financial services are applying existing frameworks to AI:

- **Model risk management (SR 11-7 / SS1/23)**: AI models used in credit, pricing, or risk decisions are in scope for model risk management — validation, documentation, ongoing monitoring
- **Explainability**: Algorithmic decisions affecting customers must be explainable — customers can request reasons for adverse decisions
- **Consumer Duty (UK)**: Firms must evidence that AI features deliver good outcomes for customers — this includes monitoring for discriminatory patterns

**Practical implication**: AI in a financial services context almost always requires a model risk management process, which means documentation, validation testing, and a second-line review before deployment.

### Healthcare (NHS / MHRA in UK, FDA in US, MDR in EU)

AI tools that assist in medical diagnosis, treatment planning, or patient triage are medical devices under UK MDR and EU MDR. This triggers a conformity assessment and UKCA/CE marking process.

AI tools used in clinical administration (appointment booking, documentation assistance, query handling) are not medical devices, but GDPR's special category provisions for health data apply to any system that processes it.

### Public Sector

Public sector bodies have additional obligations:
- **Public sector equality duty**: must actively assess and mitigate AI-driven bias in services
- **Freedom of Information**: AI-generated content and decisions may be subject to FOI requests — ensure audit trails exist
- **Procurement**: public sector AI procurement is increasingly subject to mandatory algorithmic impact assessments

---

## Practical Compliance Workflow

For each new AI feature, run through this sequence:

**Step 1 — Classify under EU AI Act**
- Does the feature make or influence decisions about individuals? → Potentially high-risk; check the Article 6 list
- Does the feature interact with users? → Limited risk; disclosure required
- Is the feature internal tooling with no user-facing AI outputs? → Likely minimal risk

**Step 2 — Identify personal data**
- What personal data flows to the LLM?
- Is any of it special category (health, financial, ethnic origin, criminal records)?
- Is a DPA with the provider in place covering this data?

**Step 3 — Check sector requirements**
- Does your organisation operate in financial services, healthcare, or public sector?
- Does the AI feature touch a regulated activity?
- If yes, identify the specific framework requirements before development starts

**Step 4 — Document**
- Record the risk classification and the reasoning
- Document the intended purpose and known limitations
- Keep this with the feature documentation, not in a separate compliance repository

---

## Check Your Understanding

1. You are building a recruitment screening tool that uses an LLM to score CVs against job requirements. Under the EU AI Act, what risk tier is this, and what obligations does it impose on your organisation before you can deploy it?

2. Your company sends customer email content to an LLM to generate support ticket summaries. The emails sometimes contain health information (e.g., "my prescription"). What GDPR obligations apply, and what would you change about the system design?

3. A colleague says "we're a UK company, so the EU AI Act doesn't apply to us." How would you respond?

4. Your product serves UK financial services customers. You want to use an LLM to pre-classify customer complaints into severity tiers to route them to the right team. What regulatory considerations apply, and how would you structure the human oversight mechanism?

5. You are writing the AI feature documentation required under the EU AI Act for a customer service chatbot. What five pieces of information must this documentation include?
