# 5.04 — Responsible Deployment

## Key Insight

The responsible deployment checklist is not an afterthought — it is a deployment gate. Treating it as a checkbox to fill in after the feature is built leads to retrofitted, inadequate controls. Treating it as a design constraint that shapes the feature from the start leads to systems that are defensible, auditable, and appropriately scoped. Every AI feature going to production should be reviewed against this checklist before it ships.

---

## The Responsible Deployment Checklist

Use this checklist as a gate before deploying any AI feature to production. "Not applicable" is a valid answer only with justification. "Not done yet" is not.

### Data and Privacy

- [ ] **PII handling:** What data categories flow to the LLM? Is the legal basis for processing documented?
- [ ] **Provider terms:** Have you confirmed the provider's data processing terms (DPA) are compatible with your data classification?
- [ ] **Data residency:** Have you confirmed the model runs in an acceptable geographic region for your data?
- [ ] **RAG corpus review:** If using RAG, have you reviewed the document corpus for PII and classified information?

### Transparency and User Experience

- [ ] **Disclosure:** Does the UI identify AI-generated content? Is the AI nature of the system disclosed in the first interaction?
- [ ] **Uncertainty language:** Are responses that involve interpretation (not direct retrieval) marked with appropriate hedging?
- [ ] **Scope communication:** Do users know what the AI can and cannot help with?

### Safety and Quality

- [ ] **Fallback behaviour:** What happens when the AI returns an incorrect or out-of-scope response? Is there a human escalation path?
- [ ] **Human oversight:** Is there a human review step for high-stakes outputs (outputs sent to external parties, outputs that influence decisions)?
- [ ] **Output validation:** Are model outputs validated against expected schema before being used or displayed?
- [ ] **Error rate known:** Has the feature been tested enough to establish a baseline error rate? Is the error rate acceptable for the use case?

### Security

- [ ] **Injection mitigations:** Are user inputs and retrieved documents enclosed in XML delimiters?
- [ ] **Guardrails:** For high-risk features, is there a guardrail model screening inputs and outputs?
- [ ] **Tool permissions:** If the AI has tools, does each tool have the minimum permissions necessary?
- [ ] **API key security:** Are API keys stored in a secrets manager, not in code or config files?

### Observability and Incident Response

- [ ] **Audit trail:** Is each AI call logged with: model version, prompt version, user/session ID, timestamp, response quality metrics?
- [ ] **Monitoring:** Are you tracking response quality metrics (not just error rates)? What does a quality regression look like in your monitoring?
- [ ] **Incident response:** Do you have a documented process for disabling the AI feature if it behaves badly? Who makes that decision? How fast can it happen?
- [ ] **Rollback:** Can the AI feature be disabled or rolled back without a code deployment?

---

## EU AI Act Disclosure Requirements

The EU AI Act (in force from 2024, phased application through 2026) introduces specific obligations for AI chatbots and AI-generated content.

### Limited Risk Systems: Transparency Obligations

Most enterprise AI chatbots fall in the "limited risk" tier, which requires:

1. **Disclosure that users are interacting with an AI system** — users must be informed, except where it is obvious from context
2. **Disclosure when content is AI-generated** — particularly for deep fakes and synthetic audio/video (this is a higher standard)

**Practical implementation for a chatbot:**

```
First message to every user:
"Hi! I'm Acme's AI assistant, powered by AI technology. I can answer questions
about [scope]. For complex issues or if I make a mistake, I'll connect you to a
human team member."
```

This satisfies the disclosure requirement and sets appropriate expectations.

### High Risk Systems

If your AI system falls in the "high risk" tier (CV screening, credit scoring, medical decisions, law enforcement tools, critical infrastructure), the obligations are significantly greater:

- Technical documentation of the system
- Conformity assessment before deployment
- Human oversight mechanisms
- Accuracy, robustness, and cybersecurity requirements
- Registration in the EU AI Act database

**For most enterprise development teams, the practical action:** Document the purpose and known limitations of your AI systems now. Retrofitting documentation later is harder and more expensive.

---

## The "3% Wrong" Problem

AI systems have known error rates. A system that is 97% accurate sounds impressive until you consider:

- At 1,000 queries per day, that's 30 wrong answers per day
- Over a year, that's approximately 11,000 wrong answers
- Each wrong answer may be acted upon by a user who trusts the system

**The 3% wrong problem is not primarily a technical problem — it is a product and governance problem.** The question is not "how do we get to 0% errors?" (you cannot) but "what is the acceptable error rate for this use case, and what process exists for handling the errors that occur?"

### Responding to Known Error Rates

The response depends on the stakes of the use case:

| Use case | Error rate acceptable? | Response |
|---------|----------------------|---------|
| "What is the API port?" — documentation lookup | 3% acceptable if there's a feedback mechanism | Add "Rate this response" button; review flagged responses |
| "Should I approve this loan?" | 3% NOT acceptable | AI provides recommendation only; human makes decision |
| "Is this patient's prescription correct?" | 0.1% NOT acceptable | AI is advisory only; clinician reviews every output |
| "Classify support tickets for routing" | 5% acceptable | Monitor routing accuracy; human review if escalation rate changes |

Document your error rate acceptance decision as part of the deployment artefacts.

---

## Output Validation for High-Stakes Use Cases

For systems where incorrect outputs cause real harm, schema validation alone is insufficient. Consider:

### Content Policy Checks

For customer-facing systems, run outputs through a content classifier before returning to users:

```java
@Service
public class ContentPolicyEnforcer {

    // Simple keyword-based approach; in production, use a dedicated classifier
    private static final List<String> PROHIBITED_PATTERNS = List.of(
        "I cannot help with",  // inappropriate refusal
        "my instructions are",  // system prompt leak
        "ignore previous",  // injection success indicator
    );

    public boolean isResponseSafe(String response) {
        String lower = response.toLowerCase();
        return PROHIBITED_PATTERNS.stream().noneMatch(lower::contains);
    }
}
```

### Confidence Scoring

Instruct the model to express uncertainty explicitly:

```
System prompt addition:
"When answering based on documentation, always indicate your confidence level:
- High confidence: The answer is stated explicitly in the documentation
- Medium confidence: The answer requires inference from the documentation
- Low confidence: The answer is not clearly addressed in the documentation

Format: Begin each response with [HIGH CONFIDENCE], [MEDIUM CONFIDENCE], or [LOW CONFIDENCE]."
```

This allows downstream systems and users to calibrate their trust in the response.

---

## Limitations and Honest Uncertainty

AI systems should be designed to express uncertainty rather than filling gaps with confident-sounding hallucinations.

**RAG with explicit fallback:** "If the answer is not in the context, say: 'Information not found.'" — forces the model to acknowledge the limit of its knowledge.

**Confidence language:** Instruct the model to use hedging language ("Based on the documentation..." "This may vary by configuration...") for responses that involve interpretation.

**Scope constraints:** Narrow the domain to what the system is genuinely competent to answer; redirect out-of-scope queries:

```
System prompt:
"You answer questions about Acme Corp's internal infrastructure documentation.
If asked about anything outside this scope — competitor products, general technology
questions, personal matters — say: 'That's outside what I can help with.
For that kind of question, try [appropriate resource].'"
```

---

## Check Your Understanding

1. A developer proposes building a RAG assistant that answers questions about employee HR records (performance reviews, salary, disciplinary notes). Work through the responsible deployment checklist for this feature. Which items would you flag as blockers before deployment?

2. Your AI-powered documentation assistant occasionally confidently states incorrect information about your API (wrong field names, wrong status codes). The error rate is approximately 3%. Is this acceptable? What is your recommendation — fix the prompt, add a disclaimer, improve the RAG corpus, or remove the feature?

3. What is the minimum disclosure a customer-facing AI chatbot must provide to comply with the EU AI Act's "limited risk" transparency requirements?

4. A user reports that your support assistant told them their account would be refunded, but it will not be. No such refund policy is in the documentation. How would you investigate this, and what safeguards would prevent it recurring?

5. Your team is under pressure to ship an AI feature quickly. The PII scrubbing layer is not yet implemented. The data flowing through the feature includes customer email addresses. What is your recommendation, and how would you frame it to the business?
