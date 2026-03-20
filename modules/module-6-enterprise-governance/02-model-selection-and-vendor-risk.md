# 6.02 — Model Selection and Vendor Risk

## Key Insight

Choosing an LLM provider is a vendor risk decision, not just a technical one. The model you choose determines which data flows outside your boundary, which SLAs govern your production availability, and which commercial terms apply to your training data. Getting this wrong has compliance, commercial, and reputational consequences that are difficult to reverse once you have built a system on top of a particular provider.

---

## The Model Selection Decision

### The Model Tiers in Practice

| Tier | Examples | When to use |
|------|----------|-------------|
| **Frontier (largest)** | Claude Opus 4.6, GPT-4o, Gemini Ultra | Complex reasoning, long documents, multi-step analysis, high-stakes outputs |
| **Mid-range** | Claude Sonnet 4.6, GPT-4o mini, Gemini Flash | Production workloads, code review, RAG responses, classification |
| **Small / fast** | Claude Haiku 4.5, GPT-3.5 Turbo | High-volume, low-complexity tasks: intent classification, routing, triage |
| **Self-hosted** | Llama 3, Mistral, CodeLlama | Data residency requirements, air-gapped environments, cost at very high volume |

### The Four Axes of Model Selection

**1. Capability match**

The question is not "which model is best" but "which model is sufficient for this task." Deploying a frontier model for simple classification is expensive over-engineering. Deploying a small model for complex multi-step reasoning produces unreliable results.

Test your specific use case with representative inputs. Benchmark scores on public leaderboards do not predict performance on your specific prompts and data.

**2. Cost at scale**

Token pricing varies by an order of magnitude between tiers. Model that right before you deploy:

```
Example: Code review feature
Input prompt: ~1,000 tokens per review
Output: ~500 tokens per review
Expected volume: 10,000 reviews/month

Claude Sonnet 4.6: ~$3/1M input, ~$15/1M output
Monthly cost: (10 × $3) + (5 × $15) = $30 + $75 = ~$105/month

Claude Haiku 4.5: ~$0.25/1M input, ~$1.25/1M output
Monthly cost: (10 × $0.25) + (5 × $1.25) = $2.50 + $6.25 = ~$9/month
```

For most enterprise features, the right model is the cheapest one that reliably meets your quality bar.

**3. Latency requirements**

Smaller models are faster. User-facing features with sub-2-second latency requirements often rule out frontier models. Async batch processing can afford the extra latency.

**4. Data classification**

What data you send to the model constrains which providers you can use. Public data → any provider. Internal business data → provider must have acceptable DPA terms. Regulated personal data → provider must have sector-specific compliance (HIPAA BAA, UK DPA 2018 terms, etc.).

---

## Provider Comparison

| Dimension | Anthropic (Claude) | OpenAI (GPT) | Azure OpenAI | Self-hosted (Ollama/vLLM) |
|-----------|-------------------|--------------|--------------|--------------------------|
| **Data processing** | Not used for training by default (API) | Not used for training (API) | Enterprise DPA available | Fully on-premises |
| **SOC 2 Type II** | ✅ | ✅ | ✅ | N/A |
| **ISO 27001** | ✅ | ✅ | ✅ (via Microsoft) | N/A |
| **GDPR DPA available** | ✅ | ✅ | ✅ | N/A |
| **HIPAA BAA available** | ✅ | ✅ | ✅ | N/A |
| **UK data residency** | ❌ (US-based processing) | ❌ | ✅ (UK Azure regions) | ✅ |
| **EU data residency** | ❌ | ❌ | ✅ (EU Azure regions) | ✅ |
| **SLA** | 99.9% | 99.9% | 99.9% | Self-managed |
| **Model version pinning** | ✅ | ✅ | ✅ | ✅ |

> Note: Terms change. Verify current provider terms before procurement decisions.

### Azure OpenAI vs OpenAI API

If you are in a regulated sector or have EU/UK data residency requirements, Azure OpenAI is often the path of least resistance: same GPT models, Microsoft's enterprise DPA, and Azure region selection. The trade-off is slightly slower model releases and Azure's pricing structure.

---

## Vendor Risk Assessment

For any LLM provider you deploy to production, answer these questions before sign-off:

### Data Processing
- [ ] Have you reviewed the provider's Data Processing Agreement (DPA)?
- [ ] Does the DPA cover the data categories you will send to the API?
- [ ] Is the provider contractually prohibited from using your API data for model training?
- [ ] If you send regulated personal data, has your DPO reviewed the DPA?

### Security and Compliance
- [ ] Does the provider hold SOC 2 Type II or equivalent?
- [ ] Is the provider's security certification current (not expired)?
- [ ] What is the provider's breach notification SLA and process?
- [ ] Are API keys treated as credentials in the provider's security model?

### Availability and Resilience
- [ ] What is the provider's stated uptime SLA?
- [ ] What is your fallback if the provider is unavailable? (Graceful degradation, alternative provider, queuing)
- [ ] Are you pinning to a specific model version, or taking automatic updates?

### Commercial
- [ ] Are you on terms that include a DPA (not just the consumer terms of service)?
- [ ] Do the provider's terms permit your use case (commercial product, customer-facing)?
- [ ] What are the notice periods and data deletion obligations on termination?

### Lock-in
- [ ] How many provider-specific SDK calls are in your codebase?
- [ ] Could you migrate to an alternative provider in under a sprint?
- [ ] Are your prompts stored as provider-agnostic text, or do they use provider-specific syntax?

---

## The Lock-in Question

Vendor lock-in in AI systems is more subtle than traditional software. The risk is not just the SDK — it is the prompt engineering. If your prompts use provider-specific features (Claude's `<thinking>` tags, OpenAI's function calling syntax, Google's Gemini-specific parameters), migrating is more work than swapping the SDK.

**Mitigations:**
- Wrap provider calls behind an interface or service class — swap implementations, not calls
- Store prompts as plain text templates that do not assume a specific provider's syntax
- Test prompts against at least two providers periodically — divergence is an early warning signal
- Use the OpenAI-compatible API interface where providers offer it (reduces SDK lock-in)

---

## The Self-Hosted Option

Self-hosted models (Llama 3, Mistral, Phi-3) are increasingly competitive for many enterprise use cases. The primary drivers for self-hosting are data residency (nothing leaves your network) and cost at very high volume.

The trade-offs:

| Factor | Managed API | Self-hosted |
|--------|-------------|-------------|
| Setup complexity | Low | High |
| Operational burden | None | Significant (GPU infra, model updates, scaling) |
| Data residency | Provider-dependent | Fully on-premises |
| Model quality at frontier | ✅ | Not yet matching GPT-4 / Claude Opus |
| Cost at scale (>10M tokens/day) | Expensive | Can be cheaper |
| Compliance | DPA-dependent | Your own controls |

For most development teams deploying their first AI features, managed APIs are the right starting point. Self-hosting makes sense when you have volume, a dedicated ML/platform team, and specific data residency requirements that managed providers cannot meet.

---

## Check Your Understanding

1. Your company is building an AI feature for a UK financial services product. You want to use GPT-4o. What vendor risk questions do you need to answer before you can proceed, and who in your organisation needs to be involved?

2. You are reviewing a colleague's code and notice they have hard-coded 47 calls to `anthropic.messages.create()` spread across the codebase. What risk does this create, and what would you recommend?

3. Build a cost model for the following feature: a RAG assistant that answers developer questions. Assume 500 queries per day, average input of 2,000 tokens (prompt + context), average output of 300 tokens. Calculate the monthly cost for Claude Sonnet 4.6 and Claude Haiku 4.5. Which would you choose and why?

4. Your legal team says you cannot send customer names or email addresses to a US-based LLM provider. List three technical approaches that would allow you to build the AI feature while complying with this constraint.

5. A self-hosted Llama 3 deployment is proposed to eliminate provider cost and data residency concerns. Your team has one DevOps engineer and no GPU infrastructure. What questions would you ask before agreeing to this approach?
