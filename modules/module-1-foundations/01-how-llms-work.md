# 1.01 — How LLMs Work

## Key Insight

A large language model is not a database, a search engine, or a reasoning engine in the human sense. It is a statistical pattern-completion system trained on vast quantities of text. Understanding this distinction — and what it implies about when LLMs succeed and when they fail — is the foundation of effective prompt engineering. Every reliable technique in this programme is a consequence of this architecture.

---

## The "Stochastic Parrot vs. Reasoning Engine" Framing

Before explaining the mechanics, it is worth addressing the most common misconception about LLMs.

**The stochastic parrot argument** holds that LLMs are sophisticated pattern-matching machines that produce plausible-sounding text without any genuine understanding. They "parrot" patterns from training data, weighted by statistical probability.

**The reasoning engine argument** holds that something more is happening: models that can solve novel mathematical problems, write working code in languages they were not explicitly trained on, and make inferences that were not present in training data appear to be doing something beyond pattern completion.

The practical position is: **it does not matter which is philosophically correct**. What matters for prompt engineering is the observable behaviour:

- LLMs generalise beyond their training data in ways that are useful
- LLMs also confabulate (hallucinate) in ways that are predictable and partially preventable
- LLMs are non-deterministic — the same prompt does not always produce the same output
- LLMs have no persistent state between calls

Treating an LLM like a database (expecting exact, reliable retrieval) leads to frustration. Treating it like a creative probabilistic engine that needs structural guardrails leads to useful systems.

---

## How LLMs Process Input: The Technical Picture

When you send a prompt to an LLM, the following happens:

### Step 1: Tokenisation

The text is broken into **tokens** — sub-word units, not words. The tokeniser splits text at character boundaries using a vocabulary of roughly 50,000–100,000 tokens.

- "Spring Boot" → two tokens: `Spring`, `Boot`
- "unrecognisable" → possibly four tokens: `un`, `recogn`, `is`, `able`
- A code snippet is tokenised character-by-character in ways that differ from natural language

**Practical implications:**

- Token count determines cost: most LLM APIs charge per input token + per output token
- Token count determines what fits in the context window
- Code and JSON are typically more token-dense than prose — a 100-line Java file might be 400–600 tokens

**Rule of thumb:** 1 token ≈ 0.75 English words. A page of prose ≈ 500 tokens.

### Step 2: Embedding

Each token is mapped to a high-dimensional vector (its **embedding**) — a list of floating-point numbers that encodes the token's semantic meaning in a learned vector space. Tokens with related meanings have numerically similar embeddings.

This is what allows the model to understand that "automobile" and "car" are related without being told explicitly. The relationship is encoded geometrically in the embedding space.

### Step 3: Attention

The model processes the full sequence of token embeddings through multiple layers of **attention**. In each layer, each token "attends" to other tokens in the sequence, computing a weighted combination of their information.

You do not need to understand the mathematics. The conceptual point is:

- Attention allows the model to identify which parts of the input are relevant to each other
- In "The dog chased the cat because it was afraid", attention is what connects "it" to "cat" rather than "dog"
- The attention mechanism is why context matters — nearby tokens and syntactically relevant tokens exert more influence than distant, unrelated ones

### Step 4: Next-Token Prediction

The model outputs a probability distribution over all tokens in its vocabulary. It samples from this distribution to select the next token. This process repeats until:

- The model produces a stop token
- The `max_tokens` limit is reached
- A stop sequence is matched

**This is the source of non-determinism.** Even with `temperature=0` (selecting the highest-probability token each time), minor floating-point variations across hardware can produce different outputs. At `temperature > 0`, sampling from the distribution introduces deliberate randomness.

---

## Why Models Hallucinate

Hallucination — the production of confident, plausible-sounding but factually incorrect output — is a direct consequence of the architecture:

1. The model is trained to produce *plausible continuations*, not *accurate statements*
2. When the model has no relevant training signal for a specific fact, it generates a plausible-sounding continuation based on adjacent patterns
3. The model has no mechanism to "know what it doesn't know" in the way a database does

**Common hallucination patterns:**

- Fabricated citations: the model has seen many citation formats and can generate structurally correct citations to papers that do not exist
- Wrong version numbers: the model has seen many version numbers and conflates them
- Invented API methods: the model knows the naming conventions of a library and invents plausibly-named methods that do not exist
- Plausible but wrong code: the model generates code that compiles but has logic errors

**What prompt engineering can do about it:**

- RAG (Module 2) grounds the model in real documents rather than training memory
- Chain-of-Thought (section 03 of this module) forces the model to reason step-by-step, which surfaces inconsistencies
- Output validation (Module 5) catches hallucinated outputs that violate structural constraints
- Explicit "say you don't know" instructions reduce (but do not eliminate) confident hallucinations

---

## Why Models Are Non-Deterministic

The same prompt sent twice will often produce slightly different outputs. This is not a bug — it is the consequence of sampling from a probability distribution.

**Temperature** controls how much randomness is introduced:

| Temperature | Effect | Use case |
|------------|--------|---------|
| 0.0 | Always select the highest-probability token | Structured outputs, code, JSON |
| 0.3–0.5 | Low randomness — predictable but not rigid | Summarisation, classification |
| 0.7–1.0 | High randomness — creative and varied | Creative writing, brainstorming |

**For production AI systems**, use `temperature=0` or close to it for structured, deterministic tasks. Non-determinism is desirable for creative tasks; it is a reliability problem for systems that need to return consistent answers to the same question.

Even at `temperature=0`, outputs can vary. Design your systems to be robust to minor variation: validate structure, not exact phrasing.

---

## Model Sizes, Pre-training, and Fine-tuning

LLMs come in different sizes, trained in different ways:

| Stage | What it is | Why it matters |
|-------|-----------|----------------|
| **Pre-training** | The model is trained on hundreds of billions of tokens of text from the internet, books, and code | This is what gives the model broad language and reasoning capability |
| **Instruction fine-tuning** | The pre-trained model is further trained on human-written instruction/response pairs | This is what makes a "raw" model into a helpful assistant |
| **RLHF** (Reinforcement Learning from Human Feedback) | Human raters score model outputs; the model is trained to maximise their scores | This is what makes the model helpful, harmless, and honest |

For prompt engineering, what matters is: **you are working with a model that has already been shaped by its training pipeline.** A model trained on primarily English text will be weaker on less-represented languages. A model instruction-tuned to be helpful will be more likely to refuse harmful requests. A model trained on code will understand code better.

You cannot change the training. You can work with the model's strengths and design around its weaknesses.

---

## Context Windows: The Working Memory Limit

Every LLM has a **context window** — the maximum number of tokens it can process in a single request (input + output combined).

| Model family | Approximate context window |
|-------------|--------------------------|
| Claude 3.5 Sonnet | 200,000 tokens |
| GPT-4o | 128,000 tokens |
| Llama 3 (70B) | 128,000 tokens |
| Smaller/cheaper models | 4,000–32,000 tokens |

The context window is the LLM's working memory. It knows nothing outside of what is in this window on each call. It has no persistent state between calls.

**Practical ceiling:** Even with large context windows, filling them entirely is slow and expensive. Effective systems use the context window efficiently, placing the most important information in the most attended positions. (See Module 2 for the Lost in the Middle effect.)

---

## Check Your Understanding

1. A colleague says "the model remembered what I told it last week." Explain why this is not technically accurate and what is actually happening when a chat interface appears to "remember" previous conversations.

2. Your code review service occasionally tells a developer that Spring Boot has a method called `@TransactionBoundary` that does not exist. What aspect of LLM architecture explains this, and what engineering approaches would reduce it?

3. Your team is deciding between two prompts for a support ticket classifier: one that forces `temperature=0` and one that uses `temperature=0.7`. What is the argument for each, and which would you recommend for this use case?

4. A manager asks why you need to send the entire conversation history on every API call rather than just the new message. Explain the stateless nature of LLM APIs in terms a non-technical manager would understand.

5. A developer reports that their prompt "works fine in testing but produces different answers in production." Given what you know about model non-determinism, what three things would you check?
