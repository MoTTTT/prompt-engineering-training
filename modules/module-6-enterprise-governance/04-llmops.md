# 6.04 — LLMOps: Running AI Systems in Production

## Key Insight

AI systems degrade in ways that traditional software does not. A service that passes all its tests today can produce measurably worse outputs next month — not because your code changed, but because the model was updated, your data distribution shifted, or the prompt that worked in December no longer works in March. LLMOps is the practice of detecting, diagnosing, and responding to this kind of drift before it becomes a customer-facing problem.

---

## MLOps vs LLMOps — What Is Different

Traditional MLOps (managing machine learning pipelines, training runs, and model deployments) is necessary but not sufficient for LLM applications. The differences matter:

| Concern | Traditional MLOps | LLMOps |
|---------|------------------|--------|
| Model retraining | Triggered by data drift or performance degradation | Largely not applicable (you use the provider's model) |
| Model versioning | You control the model version | Provider controls the model; you pin a version string |
| Evaluation | Offline metrics (accuracy, F1) on labelled dataset | Semantic evaluation; LLM-as-a-Judge; human review of samples |
| Prompt as artefact | N/A | Prompts are versioned code assets; prompt changes are deployments |
| Failure modes | Prediction errors | Hallucination, injection, tone drift, format breakage, refusals |
| Observability | Loss curves, prediction distributions | Per-request cost, latency, token counts, quality scores |
| Incident type | Model prediction wrong | Prompt injection, inappropriate output, cost spike, refusal loop |

---

## Prompt Drift

Prompt drift is the phenomenon where a prompt that works reliably today degrades over time. Causes:

1. **Model updates**: Providers release new model versions. Even "minor" updates change the model's response characteristics. If you are not pinning a version, you are implicitly accepting these changes.

2. **Data distribution shift**: The inputs your system receives in production drift from the inputs you tested against. Your prompt was tuned for the test set, not for the distribution you eventually get.

3. **Prompt-context interaction**: As your RAG corpus is updated, retrieval results change. The same user query may retrieve different context next month than it did at launch.

4. **Emergent failures**: At low volume, rare failure modes do not surface. At high volume, 0.1% failure rates matter.

### Detecting Prompt Drift

Build these checks into your monitoring pipeline:

```python
# Example: track per-request quality scores over time
import anthropic
import json
from datetime import datetime

client = anthropic.Anthropic()

def evaluate_response_quality(user_input: str, ai_response: str, expected_format: str) -> dict:
    """Use LLM-as-a-Judge to score response quality."""
    judge_prompt = f"""Score the following AI response on a scale of 1-5 for each criterion.

User input: {user_input}
AI response: {ai_response}
Expected format: {expected_format}

Return JSON: {{"relevance": 1-5, "format_compliance": 1-5, "accuracy": 1-5, "reasoning": "..."}}"""

    result = client.messages.create(
        model="claude-haiku-4-5-20251001",
        max_tokens=256,
        messages=[{"role": "user", "content": judge_prompt}]
    )
    scores = json.loads(result.content[0].text)
    scores["timestamp"] = datetime.utcnow().isoformat()
    return scores
```

Run this on a sample of production requests (1-5%) and track the score distributions over time. A drop in average score is an early warning.

---

## Model Version Pinning

Never use a provider's `latest` alias in production. Always pin to a specific model version.

### Java (Spring AI)

```java
// application.yml
spring:
  ai:
    anthropic:
      chat:
        options:
          model: claude-sonnet-4-6  # pinned — never 'latest'
          max-tokens: 1024
          temperature: 0.0
```

### Python (Anthropic SDK)

```python
import anthropic

client = anthropic.Anthropic()

# Always specify the full model ID
response = client.messages.create(
    model="claude-sonnet-4-6",  # pinned version
    max_tokens=1024,
    messages=[{"role": "user", "content": user_message}]
)
```

### Node.js / TypeScript

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

const response = await client.messages.create({
  model: "claude-sonnet-4-6",  // pinned version
  max_tokens: 1024,
  messages: [{ role: "user", content: userMessage }],
});
```

**Version migration strategy**: When a new model version is available, test it against your golden dataset (Module 4.02) before promoting to production. Treat a model version upgrade the same as a dependency upgrade — it requires evaluation, not just a version bump.

---

## Production Monitoring

### What to Track Per Request

| Metric | Why it matters | Alert threshold |
|--------|---------------|-----------------|
| **Latency (p50, p95, p99)** | User experience; detect model degradation | > 2× baseline p95 |
| **Token count (input/output)** | Cost control; prompt bloat detection | Input > 2× baseline average |
| **Cost per request** | Budget management | Daily spend > N% over 7-day average |
| **Error rate** | Provider availability; malformed output | > 1% over 5-min window |
| **Quality score (sampled)** | Output degradation | Rolling average < 3.5/5 |
| **Refusal rate** | Prompt triggering safety filters | > 0.1% unexpected refusals |
| **JSON parse failure rate** | Output format degradation | > 0.5% parse failures |

### Structured Logging for AI Requests

Log enough to diagnose incidents without exposing sensitive content:

```python
import logging
import time
import hashlib
from anthropic import Anthropic

logger = logging.getLogger(__name__)
client = Anthropic()

def ai_request_with_logging(
    prompt: str,
    user_id: str,
    feature: str
) -> str:
    start_time = time.time()
    prompt_hash = hashlib.sha256(prompt.encode()).hexdigest()[:16]  # for dedup/debug, not PII

    try:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=1024,
            messages=[{"role": "user", "content": prompt}]
        )

        latency_ms = int((time.time() - start_time) * 1000)
        output_text = response.content[0].text

        logger.info({
            "event": "ai_request",
            "feature": feature,
            "user_id": user_id,         # for audit, not content
            "prompt_hash": prompt_hash,  # for debugging, not full prompt
            "model": response.model,
            "input_tokens": response.usage.input_tokens,
            "output_tokens": response.usage.output_tokens,
            "latency_ms": latency_ms,
            "stop_reason": response.stop_reason,
        })
        return output_text

    except Exception as e:
        latency_ms = int((time.time() - start_time) * 1000)
        logger.error({
            "event": "ai_request_error",
            "feature": feature,
            "error_type": type(e).__name__,
            "latency_ms": latency_ms,
        })
        raise
```

Key principle: log metadata (token counts, latency, model version, feature name, error types) but **not** the full prompt or response content unless you have a documented legal basis and appropriate controls. Full content logging creates a PII liability.

---

## Incident Response for AI Systems

### Types of AI Incidents

| Type | Example | Severity |
|------|---------|----------|
| **Harmful output** | Chatbot produces discriminatory or harmful content | Critical |
| **Prompt injection** | Attacker manipulates system prompt behaviour | Critical |
| **PII leakage** | System prompt or RAG context exposes another user's data | Critical |
| **Hallucination causing action** | AI confidently gives wrong information that leads to a real-world error | High |
| **Cost spike** | Unexpected token usage → large unexpected bill | High |
| **Quality degradation** | Output quality drops below acceptable level | Medium |
| **Provider outage** | LLM API unavailable; feature disabled | Medium |

### Incident Response Runbook Template

**P1/P2 — Harmful output or injection:**
1. Disable the AI feature immediately (feature flag or emergency deploy)
2. Preserve logs for the incident period (do not allow rotation until reviewed)
3. Notify security/legal within 1 hour if PII is involved
4. Root cause analysis: retrieve the triggering inputs from logs (using prompt hash to find records)
5. Reproduce in non-production and implement fix before re-enabling
6. Post-incident review within 5 business days

**P3 — Quality degradation:**
1. Alert fires when rolling quality score drops below threshold
2. Check if a model version was updated (provider changelog)
3. Run the prompt against the golden dataset
4. If a model update is the cause: pin to previous version and re-evaluate new version
5. If data drift: analyse the distribution of recent inputs vs test set

---

## Dataset Versioning for Evaluation

Your golden dataset (the set of inputs and expected outputs used to evaluate prompt quality) is an artefact that needs version control as much as your code.

**What to version:**
- The input examples (user queries, code snippets, documents)
- The expected outputs (or expected properties of outputs)
- The evaluation criteria used to score them
- The date the dataset was created and when it was last validated

**Minimum structure:**

```
evaluation/
  golden-dataset-v1.jsonl     # input/expected pairs
  golden-dataset-v1.schema.json  # schema for validation
  evaluation-criteria.md      # what counts as correct
  CHANGELOG.md                # when and why the dataset changed
```

When you update your prompt, run the new prompt against the current golden dataset and record the scores. This creates a comparison baseline across prompt versions — essential for catching regressions.

---

## Check Your Understanding

1. Your AI feature has been in production for three months. Response quality has subtly degraded — not broken, but noticeably worse. You have no production monitoring in place. Walk through the steps you would take to diagnose the cause.

2. Your application uses `model: "gpt-4-latest"` in production. A provider update rolls out overnight and your JSON parsing failure rate spikes from 0.1% to 3.5%. Why did this happen and what would you change going forward?

3. You receive a customer report that your support chatbot gave a customer incorrect information about their refund policy. You have request logs with prompt hashes and token counts but not full prompt content. Can you investigate this incident? What can you determine, and what can you not?

4. Design a monitoring alert strategy for an AI feature that processes 5,000 requests per day. What metrics would you track, what thresholds would you set, and what would trigger a P1 vs P3 incident?

5. A colleague argues that the quality evaluation overhead (running LLM-as-a-Judge on 5% of requests) costs an extra $200/month and is not worth it for an internal tool. How would you respond?
