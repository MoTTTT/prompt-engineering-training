# 1.03 — Reasoning Patterns

## Key Insight

LLMs produce better answers when they are required to show their work. Chain-of-Thought prompting — instructing the model to reason step-by-step before concluding — consistently improves accuracy on logic, debugging, and multi-step tasks. The improvement is not cosmetic: making the model articulate intermediate reasoning forces it to surface inconsistencies it would otherwise gloss over. This section covers the three patterns that matter most in production AI systems.

---

## Pattern 1: Chain-of-Thought (CoT)

Chain-of-Thought prompting instructs the model to reason through a problem explicitly before giving its final answer. The key implementation detail is **using a named tag** for the reasoning block. This makes the reasoning visible, separable, and auditable.

### The Named Tag Pattern

```
Before providing your final JSON review, outline your reasoning inside
<thought_process> tags. Identify potential edge cases, exceptions, and
security issues you considered but may have dismissed.
```

The model produces output like:

```
<thought_process>
Looking at the method signature: getUser(String id) — the id parameter is not validated.
The method calls repo.findById(id) which returns Optional<User> in Spring Data JPA, but
the method signature returns User — this will throw NoSuchElementException if the user
doesn't exist, which is an unhandled exception.

There is no @Transactional annotation, which may cause issues if the caller needs
lazy-loaded associations.

No authentication check — any caller can retrieve any user by ID, which is a potential
information disclosure vulnerability if IDs are guessable.
</thought_process>

{
  "severity": "HIGH",
  "issues": [
    {"type": "MISSING_VALIDATION", "description": "id parameter is not validated", "recommendation": "Add @NotBlank validation"},
    {"type": "UNHANDLED_EXCEPTION", "description": "findById returns Optional but method signature uses User directly", "recommendation": "Use findById(id).orElseThrow(() -> new UserNotFoundException(id))"},
    {"type": "MISSING_AUTH", "description": "No authentication or authorisation check", "recommendation": "Add @PreAuthorize or explicit security check"}
  ]
}
```

**Why this works:**

- The model cannot skip over a problem once it has committed it to the `<thought_process>` block
- Errors in reasoning are visible — you can see *why* the model reached a conclusion
- The `<thought_process>` block can be logged separately and used for debugging and audit
- When reasoning and conclusion disagree, the reasoning block reveals the discrepancy

### When to Use CoT

| Good use case | Weak use case |
|---------------|--------------|
| Code review / debugging | Simple classification |
| Security analysis | Format conversion |
| Multi-step calculation | Direct lookup |
| Document comparison | Short summarisation |
| Root cause analysis | Single-fact retrieval |

CoT adds tokens (cost) and latency. Use it where reasoning quality matters, not for every task.

---

## Pattern 2: Few-Shot Prompting

Few-shot prompting provides two or three high-quality input/output examples within the prompt. The examples teach the model the exact pattern you expect — more reliably than describing the format in prose alone.

### Basic Few-Shot Structure

```
[TASK DESCRIPTION]

Examples:
Input: [EXAMPLE 1 INPUT]
Output: [EXAMPLE 1 OUTPUT]

Input: [EXAMPLE 2 INPUT]
Output: [EXAMPLE 2 OUTPUT]

Input: [EXAMPLE 3 INPUT]
Output: [EXAMPLE 3 OUTPUT]

Now process:
Input: {user_input}
Output:
```

### Worked Example: Support Ticket Classification

Without few-shot examples, the model may classify ambiguously-worded tickets inconsistently. With examples anchoring the boundary cases:

```
Classify the following support ticket into exactly one category: BILLING, TECHNICAL, ACCOUNT, OTHER.
Respond with the category label only. No explanation.

Examples:
Input: "My invoice shows the wrong amount for last month"
Output: BILLING

Input: "I cannot log in to the dashboard — the button just spins"
Output: ACCOUNT

Input: "The API is returning 500 errors intermittently since 14:00 UTC"
Output: TECHNICAL

Input: "Do you have a mobile app?"
Output: OTHER

Now classify:
Input: {ticket_text}
Output:
```

### Choosing Examples

- **Include boundary cases:** Examples that could go either way are more valuable than obvious cases
- **Cover every output class:** At least one example per category anchors the model's understanding
- **Two to five examples is the sweet spot:** More adds tokens without proportional benefit
- **Examples should match the style of real inputs:** If real tickets use informal language, use informal language in examples

### Few-Shot for JSON Output

Few-shot is particularly effective for complex JSON schemas where prose description is ambiguous:

```
Generate a structured code review in the following JSON format.

Example 1:
Code: public void deleteUser(Long id) { userRepo.delete(id); }
Review: {"severity": "HIGH", "issues": [{"type": "MISSING_AUTH", "description": "No authorisation check before deletion", "recommendation": "Add @PreAuthorize(\"hasRole('ADMIN')\")"}]}

Example 2:
Code: public String getVersion() { return "1.0.0"; }
Review: {"severity": "NONE", "issues": []}

Now review:
Code: {code_to_review}
Review:
```

---

## Pattern 3: Generate → Critique → Revise (Self-Criticism Loop)

This is the pattern the Day 1 slide deck calls "self-criticism loops" and elsewhere appears as "iterative refinement." It is a **multi-call workflow** in which:

1. **Generate:** The model produces a first draft
2. **Critique:** A second call evaluates the draft against specific criteria
3. **Revise:** A third call produces an improved version based on the critique

This pattern is more expensive (three LLM calls) but produces noticeably higher-quality output for high-stakes content.

### Worked Example: Architecture Decision Record

**Call 1 — Generate:**

```
[SYSTEM]
You are a Senior Software Architect. Write an Architecture Decision Record (ADR)
for the following decision.

[USER]
Decision: Use event sourcing for the order management service.
Context: We need audit trail, temporal queries, and the ability to replay events.
Constraints: Team has no prior event sourcing experience.
```

**Call 2 — Critique:**

```
[SYSTEM]
You are a Senior Technical Editor reviewing Architecture Decision Records.

[USER]
Review the following ADR against these criteria:
- Does it clearly state the problem being solved?
- Are the alternatives considered?
- Are the trade-offs (not just benefits) clearly stated?
- Is the decision reversible, and is that addressed?
- Are the consequences of the decision spelled out?

List each criterion with PASS or FAIL and a one-sentence explanation.

<adr_to_review>
{output from Call 1}
</adr_to_review>
```

**Call 3 — Revise:**

```
[SYSTEM]
You are a Senior Software Architect.

[USER]
Revise the following ADR to address all FAIL criteria in the editorial review.
Return only the revised ADR, no meta-commentary.

Original ADR:
<original_adr>
{output from Call 1}
</original_adr>

Editorial review:
<editorial_review>
{output from Call 2}
</editorial_review>
```

### Implementing the Loop in Code

The same pattern applies to code generation, documentation, and any high-stakes text:

### Java (Spring AI)

```java
@Service
public class IterativeDocService {

    private final ChatClient chatClient;

    public String generateRefinedDoc(String spec) {
        // Call 1: Generate
        String draft = chatClient.prompt()
            .system(loadPrompt("doc-generator-system.st"))
            .user(spec)
            .call()
            .content();

        // Call 2: Critique
        String critique = chatClient.prompt()
            .system(loadPrompt("doc-critic-system.st"))
            .user("Review this document:\n<draft>\n" + draft + "\n</draft>")
            .call()
            .content();

        // Call 3: Revise
        return chatClient.prompt()
            .system(loadPrompt("doc-reviser-system.st"))
            .user("Original:\n<draft>\n" + draft + "\n</draft>\n\nCritique:\n<critique>\n" + critique + "\n</critique>")
            .call()
            .content();
    }
}
```

### Python (Anthropic SDK)

```python
import anthropic

client = anthropic.Anthropic()

def generate_refined_doc(spec: str) -> str:
    # Call 1: Generate
    draft_response = client.messages.create(
        model="claude-opus-4-5",
        max_tokens=1024,
        system="You are a Senior Software Architect. Write clear, complete ADRs.",
        messages=[{"role": "user", "content": spec}]
    )
    draft = draft_response.content[0].text

    # Call 2: Critique
    critique_response = client.messages.create(
        model="claude-haiku-4-5",  # cheaper model for critique step
        max_tokens=512,
        system="You are a Senior Technical Editor. Identify gaps and weaknesses in ADRs.",
        messages=[{"role": "user", "content": f"Review this ADR:\n<draft>\n{draft}\n</draft>"}]
    )
    critique = critique_response.content[0].text

    # Call 3: Revise
    revised_response = client.messages.create(
        model="claude-opus-4-5",
        max_tokens=1024,
        system="You are a Senior Software Architect. Revise documents to address editorial feedback.",
        messages=[{
            "role": "user",
            "content": f"Original:\n<draft>\n{draft}\n</draft>\n\nCritique:\n<critique>\n{critique}\n</critique>\n\nReturn only the revised document."
        }]
    )
    return revised_response.content[0].text
```

### Node.js / TypeScript (@anthropic-ai/sdk)

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

async function generateRefinedDoc(spec: string): Promise<string> {
  // Call 1: Generate
  const draftResponse = await client.messages.create({
    model: "claude-opus-4-5",
    max_tokens: 1024,
    system: "You are a Senior Software Architect. Write clear, complete ADRs.",
    messages: [{ role: "user", content: spec }],
  });
  const draft = (draftResponse.content[0] as { text: string }).text;

  // Call 2: Critique (cheaper model)
  const critiqueResponse = await client.messages.create({
    model: "claude-haiku-4-5",
    max_tokens: 512,
    system:
      "You are a Senior Technical Editor. Identify gaps and weaknesses in ADRs.",
    messages: [
      {
        role: "user",
        content: `Review this ADR:\n<draft>\n${draft}\n</draft>`,
      },
    ],
  });
  const critique = (critiqueResponse.content[0] as { text: string }).text;

  // Call 3: Revise
  const revisedResponse = await client.messages.create({
    model: "claude-opus-4-5",
    max_tokens: 1024,
    system:
      "You are a Senior Software Architect. Revise documents to address editorial feedback.",
    messages: [
      {
        role: "user",
        content: `Original:\n<draft>\n${draft}\n</draft>\n\nCritique:\n<critique>\n${critique}\n</critique>\n\nReturn only the revised document.`,
      },
    ],
  });
  return (revisedResponse.content[0] as { text: string }).text;
}
```

### Cost Management for Multi-Call Patterns

Using a cheaper model for the critique step (as shown in the Python and TypeScript examples above) reduces cost significantly while preserving quality where it matters most: the generation and revision steps that produce the final output.

---

## Combining the Patterns

The patterns compose:

```
[SYSTEM PROMPT: PERSONA + CONSTRAINTS]

You are a Senior Java Developer. Review the provided code for security issues.

Before your final JSON response, write your analysis in <thought_process> tags.

Examples:
Input: [example buggy code]
Output: <thought_process>...</thought_process>{"severity": "HIGH", ...}

Input: [example clean code]
Output: <thought_process>...</thought_process>{"severity": "NONE", ...}

<code_to_review>
{user_code}
</code_to_review>

[OUTPUT INDICATOR]
Respond with <thought_process> block then valid JSON matching the schema above.
```

This prompt uses: Persona pattern, XML delimiters, Chain-of-Thought (named tag), and Few-Shot. This is exactly what Lab 01 implements.

---

## Check Your Understanding

1. You add a `<thought_process>` block to your code review prompt. The model now writes extensive reasoning but the final JSON is still wrong. What does this tell you, and what would you try next?

2. You need to classify customer emails into 12 categories. You have 24 labelled examples. How many would you use as few-shot examples in the prompt, and how would you choose which ones?

3. The Generate → Critique → Revise pattern costs three LLM calls instead of one. For what types of content in your organisation would this additional cost be justified? Give two concrete examples.

4. A colleague uses zero-shot (no examples) for a complex JSON generation task. The output format is inconsistent across calls. They argue that "the model should be smart enough to follow the schema without examples." How would you respond?

5. You are building a self-criticism loop for generating API documentation. Write the critique prompt: what criteria should it check, and how should it format its output so the revise step can act on it?
