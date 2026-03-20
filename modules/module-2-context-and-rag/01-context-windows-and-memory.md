# 2.01 — Context Windows and Memory

## Key Insight

An LLM has no persistent memory. Between API calls, it knows nothing. Within a single API call, everything it "knows" is what you put into the prompt. The context window is the LLM's working memory — finite, expensive, and subject to a failure mode called the Lost in the Middle effect. Managing context well is the difference between an assistant that degrades after a few exchanges and one that remains reliable across long sessions.

---

## The Context Window

Every LLM has a maximum context window — the total number of tokens it can process in a single request (input + output combined).

| Model family | Approximate context window |
|-------------|--------------------------|
| Claude 3.5 Sonnet | 200,000 tokens |
| GPT-4o | 128,000 tokens |
| Llama 3 (70B) | 128,000 tokens |
| Smaller/cheaper models | 4,000–32,000 tokens |

**Practical ceiling:** Even with a 200K token window, filling it entirely is slow and expensive. A 200K token context with a frontier model can cost $0.30–$3.00 per call depending on the model. At enterprise scale, context window management is a cost control tool, not just a technical constraint.

---

## The Lost in the Middle Effect

Research has consistently shown that LLMs attend most strongly to:

- Content at the **beginning** of the prompt (system prompt and early instructions)
- Content at the **end** of the prompt (the most recent user message)

Content placed in the **middle** of a long prompt — including critical context, retrieved documents, or important constraints — tends to be underweighted or ignored. This is called the **Lost in the Middle effect**.

**This matters most in RAG.** When you inject retrieved documents into a prompt, the documents in the middle of the context block receive less attention than those at the start and end. If the critical answer is in the third of five retrieved chunks and you place them in order of retrieval, the critical chunk may be in the "lost" zone.

**Implications for design:**

- Place the most important instructions at the top (system prompt) and at the bottom (just before asking the question)
- Do not bury key constraints in the middle of a long document block
- When using RAG, experiment with ordering: most relevant chunk first, or most relevant chunk last
- For long RAG contexts, consider splitting into multiple calls rather than one giant context

---

## Context Budget Planning

For any RAG system, plan the context budget explicitly:

| Component | Typical size | Notes |
|-----------|-------------|-------|
| System prompt | 200–800 tokens | Keep lean; audit regularly |
| Conversation history | 0–5,000 tokens | Depends on session length and compression strategy |
| Retrieved context (RAG) | 1,500–8,000 tokens | Main variable; topK × chunk_size |
| User question | 20–200 tokens | Usually small |
| Reserved for output | 500–2,000 tokens | Set max_tokens to this value |
| **Total** | **2,200–16,000 tokens** | **Leaves substantial headroom in modern context windows** |

The implication: even with large context windows, most RAG systems can operate efficiently in a small fraction of the available context, preserving cost efficiency.

---

## System Prompts and Session Initialisation

The **system prompt** is the highest-trust layer of the prompt. It:

- Sets the persona and behavioural constraints
- Defines output format expectations
- Establishes what the model should and should not do
- Persists as the first message in every conversation turn

In production systems, the system prompt is fixed and controlled by the application developer. User input is passed separately as a user message. This separation is both an architectural and a security boundary (see Module 5).

**Example session structure:**

```
[System] You are an internal Support Assistant for Acme Corp.
         Answer only based on the provided documentation context.
         If the answer is not in the context, say: "Information not found."
         Format all responses as plain prose. Do not use bullet points.

[User] How do I reset my VPN credentials?

[Assistant] To reset your VPN credentials, navigate to...

[User] What about SSH keys?
```

Each exchange appends to the conversation history. The model sees the full history on each turn.

---

## Memory Architectures

For stateful applications, memory must be maintained explicitly at the application layer. Common patterns:

| Pattern | Mechanism | Use case | Trade-off |
|---------|-----------|---------|----------|
| **In-context history** | Pass all prior turns in the API call | Short sessions, simple assistants | High token cost for long sessions |
| **Summarised memory** | Compress history into a summary, inject as context | Long sessions, cost-sensitive | Some information loss in compression |
| **External memory store** | Store facts/events in a database, retrieve on demand | Persistent user profiles, enterprise knowledge | More complex; retrieval adds latency |
| **Episodic memory** | Store significant events/decisions, not full transcripts | Agents that need to recall past actions | Requires deciding what is "significant" |

### Stateless: Each Call Is Independent

Most simple AI features do not need memory at all. A code reviewer, a document summariser, or a single-question support bot can be stateless — each API call is independent. This is the simplest and cheapest approach.

### In-Context History: The Simple Case

```python
conversation_history = []

def chat(user_message: str) -> str:
    conversation_history.append({"role": "user", "content": user_message})
    response = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=1024,
        system=SYSTEM_PROMPT,
        messages=conversation_history
    )
    assistant_message = response.content[0].text
    conversation_history.append({"role": "assistant", "content": assistant_message})
    return assistant_message
```

This works for short sessions. For sessions that grow long, implement a maximum history length: drop the oldest turns or compress them.

### Summarised Memory: Long Sessions

When history grows large, compress earlier turns:

```python
COMPRESSION_THRESHOLD = 10  # compress when history exceeds this many turns

def compress_history(history: list) -> list:
    if len(history) <= COMPRESSION_THRESHOLD:
        return history

    # Compress the oldest half of the history
    to_compress = history[:len(history)//2]
    recent = history[len(history)//2:]

    summary_response = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=256,
        messages=[{
            "role": "user",
            "content": f"Summarise the key points from this conversation in 3-5 sentences:\n{json.dumps(to_compress)}"
        }]
    )
    summary = summary_response.content[0].text

    compressed_turn = {
        "role": "user",
        "content": f"[Conversation summary: {summary}]"
    }
    return [compressed_turn] + recent
```

---

## Token Optimisation Strategies

### 1. Conversation History Pruning

As sessions grow long, history management is essential. Implement a sliding window or summarisation step. This can reduce per-call token count by 60–80% for long sessions.

### 2. System Prompt Auditing

System prompts accumulate instructions over time. Every sentence runs on every API call at scale. Audit regularly:

- Remove redundant instructions that say the same thing twice
- Remove instructions for features that were removed
- Replace verbose prose with shorter, equally effective alternatives

### 3. Output Format Constraints

Verbose output formats cost more. If you only need a severity rating and a one-sentence finding, say so explicitly. An unconstrained prompt that asks for a "review" may return 500 words when 50 would serve.

### 4. Chunk Size Tuning in RAG

In RAG pipelines, retrieved context is the largest variable cost. Tuning the chunk size, overlap, and topK parameter directly controls how many tokens are injected per call. See section 2.02 for chunking strategies.

### 5. Model Downgrade for Sub-Tasks

Not every step requires the frontier model. In multi-step workflows, classification, extraction, and formatting steps can often run on cheaper models; only the final synthesis step needs the most capable model.

---

## Check Your Understanding

1. You are building a customer support bot. After 20 exchanges, the bot starts ignoring instructions that were in its system prompt. What is the most likely cause, and what would you do to fix it?

2. A colleague proposes storing the full conversation history for every user session indefinitely. What are the technical and cost problems with this approach?

3. You are building a RAG assistant. You have 50 relevant document chunks but can only inject 5 without exceeding your token budget. How do you decide which 5 to use, and where in the prompt do you place them?

4. Your system prompt is 3,000 tokens. On investigation, you find that 2,000 of those tokens are detailed instructions for handling edge cases that have never occurred in production. What would you do, and what risk does this change carry?

5. The Lost in the Middle effect means that information in the middle of a long RAG context is underweighted. Design a context ordering strategy for a RAG assistant that retrieves 5 chunks of 600 tokens each. Where do you put the most relevant chunk?
