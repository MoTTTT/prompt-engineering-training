# 3.01 — API Fundamentals

## Key Insight

The LLM API is not a magic black box — it is a structured HTTP endpoint that accepts a JSON payload containing messages, model parameters, and optionally tool definitions. Every LLM provider (Anthropic, OpenAI, Google, Mistral) uses a similar structure. Understanding the API contract at this level gives you full control over cost, quality, and reliability. The framework you choose (Spring AI, the Anthropic SDK, the OpenAI SDK) wraps this structure but does not change it.

---

## The Message Structure

All major LLM APIs use a messages-based interface. A request contains:

```json
{
  "model": "claude-haiku-4-5",
  "system": "You are a Senior Java Developer...",
  "messages": [
    {"role": "user", "content": "Review this code: ..."},
    {"role": "assistant", "content": "I found two issues..."},
    {"role": "user", "content": "Can you suggest a fix for the first one?"}
  ],
  "max_tokens": 1024,
  "temperature": 0.0
}
```

**Key fields:**

| Field | Role | Notes |
|-------|------|-------|
| `system` | Developer-controlled instructions; highest trust level | Set once; persists across all turns |
| `messages` | Conversation history as role/content pairs | You manage this array; append on each turn |
| `max_tokens` | Hard upper limit on output length | Prevents runaway costs; set to ~2x expected response |
| `temperature` | Output randomness (0=deterministic, 1=creative) | Use 0 for structured tasks; 0.3–0.7 for creative |
| `top_p` | Nucleus sampling threshold | Leave at default unless specifically tuning |
| `stop` | Sequences that terminate generation | Useful for enforcing output structure |

The three message roles:
- `system`: Developer-controlled; sets persona, constraints, output format
- `user`: Input from the human or calling application
- `assistant`: The model's previous responses (for conversation history)

---

## API Parameters Reference

| Parameter | Effect | Typical production value |
|-----------|--------|--------------------------|
| `temperature` | 0=deterministic, 1=creative | 0–0.2 for structured outputs |
| `max_tokens` | Maximum output tokens | Set to 2× expected output to avoid truncation |
| `top_p` | Nucleus sampling threshold | Leave at default (1.0) unless tuning |
| `stop` | Stop sequences that end generation | `["\n\n", "```"]` can enforce concise output |

**Critical:** Always set `max_tokens` explicitly. Without it, the model may generate a very long response on an unconstrained prompt, creating unexpected cost and latency.

---

## SDK Setup and Basic Call

### Java (Spring AI)

Add dependencies to `build.gradle`:

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.ai:spring-ai-anthropic-spring-boot-starter'
}
```

Configure in `application.properties`:

```properties
spring.ai.anthropic.api-key=${ANTHROPIC_API_KEY}
spring.ai.anthropic.chat.options.model=claude-haiku-4-5
spring.ai.anthropic.chat.options.max-tokens=1024
spring.ai.anthropic.chat.options.temperature=0.0
```

Make a call using `ChatClient`:

```java
@Service
public class ReviewService {

    private final ChatClient chatClient;

    public ReviewService(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }

    public String reviewCode(String code) {
        return chatClient.prompt()
            .system("You are a Senior Java Developer reviewing for security issues.")
            .user("<code_to_review>" + code + "</code_to_review>")
            .call()
            .content();
    }
}
```

### Python (Anthropic SDK)

```bash
pip install anthropic
```

```python
import anthropic

client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY from environment

def review_code(code: str) -> str:
    response = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=1024,
        temperature=0.0,
        system="You are a Senior Java Developer reviewing for security issues.",
        messages=[
            {"role": "user", "content": f"<code_to_review>{code}</code_to_review>"}
        ]
    )
    return response.content[0].text
```

### Node.js / TypeScript (@anthropic-ai/sdk)

```bash
npm install @anthropic-ai/sdk
```

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic(); // reads ANTHROPIC_API_KEY from environment

async function reviewCode(code: string): Promise<string> {
  const response = await client.messages.create({
    model: "claude-haiku-4-5",
    max_tokens: 1024,
    temperature: 0,
    system: "You are a Senior Java Developer reviewing for security issues.",
    messages: [
      {
        role: "user",
        content: `<code_to_review>${code}</code_to_review>`,
      },
    ],
  });
  return (response.content[0] as { text: string }).text;
}
```

The structure is identical across all three. The difference is syntax, not concept.

---

## Prompt Templates: Prompts as Code Assets

Hard-coding prompt text inside application code is the equivalent of hard-coding SQL queries: it makes prompts impossible to version, test independently, or update without a code deployment.

**Best practice:** Store prompt templates as external resource files.

### Java (Spring AI): `.st` Template Files

Spring AI uses StringTemplate (`.st`) files stored in `src/main/resources/prompts/`:

**`reviewer-system.st`:**
```
# Version: 1.3 | Updated: 2026-03-01
# Change: Added @Transactional check
You are a Senior Java Developer and Security Auditor specialising in Spring Boot.

Review the Java code in <code_to_review> tags for:
- Security vulnerabilities
- Missing @Transactional annotations on JPA operations
- Unhandled Optionals from Spring Data
- Missing input validation

Before your final JSON, write reasoning in <thought_process> tags.

Return ONLY valid JSON:
{"severity": "HIGH|MEDIUM|LOW|NONE", "issues": [{"type": "...", "description": "...", "recommendation": "..."}]}
```

Load and use in Spring AI:

```java
@Value("classpath:prompts/reviewer-system.st")
private Resource systemPromptResource;

public String reviewCode(String code) {
    return chatClient.prompt()
        .system(systemPromptResource)
        .user("<code_to_review>" + code + "</code_to_review>")
        .call()
        .content();
}
```

### Python and Node.js: Plain Text Files

The same principle applies. Store prompts in `.txt` files and load them at startup:

```python
from pathlib import Path

# Load once at startup; cache in memory
SYSTEM_PROMPT = Path("prompts/reviewer_system.txt").read_text()

def review_code(code: str) -> str:
    response = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=1024,
        system=SYSTEM_PROMPT,
        messages=[{"role": "user", "content": f"<code_to_review>{code}</code_to_review>"}]
    )
    return response.content[0].text
```

Every prompt template should have a version comment header so you can trace which prompt version was deployed when a quality regression is reported.

---

## Conversation History

For multi-turn conversations, you maintain the message history explicitly:

### Java (Spring AI)

Spring AI's `ChatClient` can manage conversation history automatically via an `InMemoryChatMemory` adviser, or you can manage it manually:

```java
@Service
public class ConversationService {

    private final ChatClient chatClient;
    // In production, store conversation history per user, not in-memory
    private final List<Message> history = new ArrayList<>();

    public ConversationService(ChatClient.Builder builder) {
        this.chatClient = builder
            .defaultAdvisors(new MessageChatMemoryAdvisor(new InMemoryChatMemory()))
            .build();
    }

    public String chat(String userMessage) {
        return chatClient.prompt()
            .user(userMessage)
            .call()
            .content();
    }
}
```

### Python (Anthropic SDK)

```python
conversation_history = []

def chat(user_message: str) -> str:
    conversation_history.append({
        "role": "user",
        "content": user_message
    })

    response = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=1024,
        system="You are a helpful technical assistant.",
        messages=conversation_history
    )

    assistant_message = response.content[0].text
    conversation_history.append({
        "role": "assistant",
        "content": assistant_message
    })

    return assistant_message
```

---

## Structured Output

Forcing JSON output from an LLM reliably requires more than asking nicely.

### Strategy 1: Output Indicator in the Prompt

Include the exact JSON schema in the system prompt with a populated example:

```
Your response MUST be valid JSON matching this schema exactly:
{
  "severity": "HIGH|MEDIUM|LOW|NONE",
  "issues": [
    {
      "type": "string",
      "description": "string",
      "recommendation": "string"
    }
  ]
}
Example of valid response: {"severity": "HIGH", "issues": [{"type": "MISSING_AUTH", "description": "...", "recommendation": "..."}]}
```

### Strategy 2: Parse and Retry on Failure

Models occasionally produce subtly malformed JSON. Implement a retry:

```java
private ReviewResult parseWithRetry(String response) {
    try {
        return objectMapper.readValue(extractJson(response), ReviewResult.class);
    } catch (JsonProcessingException e) {
        // One retry with an explicit correction instruction
        String corrected = chatClient.prompt()
            .user("The following JSON is malformed. Fix it and return ONLY valid JSON, no other text:\n" + response)
            .call()
            .content();
        try {
            return objectMapper.readValue(extractJson(corrected), ReviewResult.class);
        } catch (JsonProcessingException retryException) {
            throw new ReviewParseException("Failed to parse review response after retry");
        }
    }
}
```

The same retry pattern applies identically in Python and TypeScript — catch the JSON parse error, send a correction call, try again once.

---

## Streaming Responses

For long responses or interactive UIs, streaming allows the client to display output as it is generated rather than waiting for the full response.

### Java (Spring AI)

```java
public Flux<String> reviewCodeStreaming(String code) {
    return chatClient.prompt()
        .system(systemPromptResource)
        .user("<code_to_review>" + code + "</code_to_review>")
        .stream()
        .content();
}
```

Wire this to a Spring WebFlux `text/event-stream` endpoint for real-time display in a browser.

### Python (Anthropic SDK)

```python
def review_code_streaming(code: str):
    with client.messages.stream(
        model="claude-haiku-4-5",
        max_tokens=1024,
        system=SYSTEM_PROMPT,
        messages=[{"role": "user", "content": f"<code_to_review>{code}</code_to_review>"}]
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)
```

### Node.js / TypeScript (@anthropic-ai/sdk)

```typescript
async function reviewCodeStreaming(code: string): Promise<void> {
  const stream = await client.messages.stream({
    model: "claude-haiku-4-5",
    max_tokens: 1024,
    system: SYSTEM_PROMPT,
    messages: [
      { role: "user", content: `<code_to_review>${code}</code_to_review>` },
    ],
  });

  for await (const chunk of stream) {
    if (
      chunk.type === "content_block_delta" &&
      chunk.delta.type === "text_delta"
    ) {
      process.stdout.write(chunk.delta.text);
    }
  }
}
```

**Note on structured output and streaming:** Streaming and JSON parsing are in tension. If you need to parse the full response as JSON, wait for the complete response. If you need streaming, parse incrementally or buffer and parse at the end.

---

## Error Handling and Retries

LLM API calls can fail for several reasons: rate limiting, network timeouts, model overload, or content policy rejection. Always handle these explicitly.

### Java (Spring AI)

```java
@Retryable(
    retryFor = {AiApiException.class},
    maxAttempts = 3,
    backoff = @Backoff(delay = 1000, multiplier = 2)
)
public String reviewCode(String code) {
    return chatClient.prompt()
        .system(systemPromptResource)
        .user("<code_to_review>" + code + "</code_to_review>")
        .call()
        .content();
}

@Recover
public String reviewCodeFallback(AiApiException e, String code) {
    log.error("Code review failed after retries: {}", e.getMessage());
    return "{\"severity\": \"ERROR\", \"issues\": [], \"error\": \"Review service unavailable\"}";
}
```

### Python (Anthropic SDK)

The Anthropic Python SDK has built-in retry with exponential backoff:

```python
# The SDK retries automatically on rate limits and transient errors
# Configure the retry behaviour at client instantiation
client = anthropic.Anthropic(
    max_retries=3,  # default is 2
)

# Handle non-retryable errors explicitly
from anthropic import APIConnectionError, RateLimitError, APIStatusError

try:
    result = review_code(code)
except RateLimitError:
    # Specific handling for rate limits
    raise
except APIConnectionError:
    # Network failure
    raise
except APIStatusError as e:
    # HTTP error (4xx, 5xx)
    print(f"API error {e.status_code}: {e.message}")
    raise
```

---

## Check Your Understanding

1. What is the difference between the `system` field and a `user` message in the API payload? Why does it matter architecturally which one carries your instructions?

2. You deploy a prompt template as a `.st` file and it works perfectly. Three months later the responses have degraded noticeably. The prompt file has not changed. What else might have changed, and how would you investigate?

3. A response from your code review service occasionally fails JSON parsing. Walk through how you would handle this failure gracefully without exposing the error to the end user.

4. Why is setting `max_tokens` in production important, even if you expect responses to be short? What could happen if you omit it?

5. The `temperature` parameter is set to `0.8` in your production code review service. A colleague says this is fine because "it makes the responses more interesting." How would you respond?
