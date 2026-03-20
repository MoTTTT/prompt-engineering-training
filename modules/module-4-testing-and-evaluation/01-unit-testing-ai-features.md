# 4.01 — Unit Testing AI Features

## Key Insight

Unit testing AI features does not mean testing the LLM. The LLM is an external service — you do not test it any more than you unit-test a database. What you unit-test is the plumbing around the LLM: that your application loads the correct prompt template, injects the right parameters, handles the response correctly, and fails gracefully when the LLM returns unexpected content. This is entirely testable without making a single real LLM call.

---

## What to Unit Test in AI Code

| What to test | Why it matters | How to test |
|-------------|----------------|------------|
| Prompt template loading | Wrong template = wrong behaviour | Assert template is loaded from correct file |
| Parameter injection | Missing params = broken prompt | Assert params are substituted correctly |
| XML delimiter enclosure | Missing delimiters = injection risk | Assert user input is wrapped in tags |
| Response JSON parsing | Parse failure = unhandled exception | Test with valid JSON, malformed JSON, empty response |
| Error handling | Model errors should not reach users | Test retry logic and fallback responses |
| Schema validation | Wrong schema = corrupted data | Test with in-schema and out-of-schema responses |

What you do NOT unit-test:
- Whether the LLM gives the "right" answer (that is semantic evaluation, covered in section 4.02)
- Whether the LLM API is available (that is an integration test)
- The model's safety guardrails (those belong to the model provider)

---

## Mocking LLM Responses

### Java (Spring AI)

In Spring AI, `ChatClient` is the component to mock. The `ChatClient.Builder` pattern makes this straightforward:

```java
@ExtendWith(MockitoExtension.class)
class CodeReviewServiceTest {

    @Mock
    private ChatClient chatClient;

    @Mock
    private ChatClient.Builder chatClientBuilder;

    @Mock
    private ChatClient.ChatClientRequest chatClientRequest;

    @Mock
    private ChatClient.ChatClientRequest.CallResponseSpec callResponseSpec;

    private CodeReviewService service;

    @BeforeEach
    void setUp() {
        when(chatClientBuilder.build()).thenReturn(chatClient);
        service = new CodeReviewService(chatClientBuilder, new ObjectMapper());
    }

    @Test
    void shouldReturnParsedReviewResult() {
        // Arrange: configure the mock to return a known JSON response
        String mockResponse = """
            <thought_process>
            Looking at the method: missing null check on the return value of findById().
            </thought_process>
            {"severity": "HIGH", "issues": [{"type": "UNHANDLED_OPTIONAL", "description": "findById may return empty", "recommendation": "Use orElseThrow()"}]}
            """;

        when(chatClient.prompt()).thenReturn(chatClientRequest);
        when(chatClientRequest.system(any(Resource.class))).thenReturn(chatClientRequest);
        when(chatClientRequest.user(anyString())).thenReturn(chatClientRequest);
        when(chatClientRequest.call()).thenReturn(callResponseSpec);
        when(callResponseSpec.content()).thenReturn(mockResponse);

        // Act
        ReviewResult result = service.review("public User getUser(String id) { return repo.findById(id); }");

        // Assert
        assertThat(result.severity()).isEqualTo("HIGH");
        assertThat(result.issues()).hasSize(1);
        assertThat(result.issues().get(0).type()).isEqualTo("UNHANDLED_OPTIONAL");
    }

    @Test
    void shouldHandleMalformedJsonFromModel() {
        // Arrange: simulate the model returning malformed JSON
        when(chatClient.prompt()).thenReturn(chatClientRequest);
        when(chatClientRequest.system(any(Resource.class))).thenReturn(chatClientRequest);
        when(chatClientRequest.user(anyString())).thenReturn(chatClientRequest);
        when(chatClientRequest.call()).thenReturn(callResponseSpec);
        when(callResponseSpec.content()).thenReturn("Here is my review: {broken json");

        // Act & Assert
        assertThatThrownBy(() -> service.review("any code"))
            .isInstanceOf(ReviewParseException.class)
            .hasMessageContaining("Failed to parse review");
    }
}
```

### Testing Prompt Template Loading

```java
@Test
void shouldLoadSystemPromptFromFile() {
    // This test verifies the template file exists and is loadable
    // It does NOT call the LLM
    Resource promptResource = new ClassPathResource("prompts/reviewer-system.st");

    assertThat(promptResource.exists())
        .as("System prompt template must exist")
        .isTrue();

    assertThatCode(() -> promptResource.getContentAsString(StandardCharsets.UTF_8))
        .doesNotThrowAnyException();
}

@Test
void shouldContainRequiredPromptElements() throws IOException {
    String promptText = new ClassPathResource("prompts/reviewer-system.st")
        .getContentAsString(StandardCharsets.UTF_8);

    // Verify the four pillars are present
    assertThat(promptText)
        .as("System prompt must contain persona instruction")
        .containsIgnoringCase("senior java developer");

    assertThat(promptText)
        .as("System prompt must require JSON output")
        .containsIgnoringCase("json");

    assertThat(promptText)
        .as("System prompt must instruct CoT")
        .contains("thought_process");
}
```

### Testing XML Delimiter Enclosure

```java
@Test
void shouldWrapCodeInXmlDelimiters() {
    // Capture the actual user message sent to the ChatClient
    ArgumentCaptor<String> userMessageCaptor = ArgumentCaptor.forClass(String.class);

    when(chatClient.prompt()).thenReturn(chatClientRequest);
    when(chatClientRequest.system(any())).thenReturn(chatClientRequest);
    when(chatClientRequest.user(userMessageCaptor.capture())).thenReturn(chatClientRequest);
    when(chatClientRequest.call()).thenReturn(callResponseSpec);
    when(callResponseSpec.content()).thenReturn("{\"severity\": \"NONE\", \"issues\": []}");

    service.review("String foo = \"bar\";");

    String capturedMessage = userMessageCaptor.getValue();
    assertThat(capturedMessage)
        .as("User input must be enclosed in XML tags for injection protection")
        .contains("<code_to_review>")
        .contains("</code_to_review>");
}
```

---

### Python (Anthropic SDK)

```python
import pytest
from unittest.mock import MagicMock, patch
import json

# Mock the Anthropic client at the module level
@pytest.fixture
def mock_anthropic(monkeypatch):
    mock_client = MagicMock()
    monkeypatch.setattr("your_module.client", mock_client)
    return mock_client

def make_mock_response(text: str):
    """Helper to build a mock Anthropic response object."""
    response = MagicMock()
    response.content = [MagicMock(text=text)]
    response.stop_reason = "end_turn"
    return response

def test_returns_parsed_review_result(mock_anthropic):
    mock_response_text = json.dumps({
        "severity": "HIGH",
        "issues": [{"type": "UNHANDLED_OPTIONAL", "description": "test", "recommendation": "fix"}]
    })
    mock_anthropic.messages.create.return_value = make_mock_response(mock_response_text)

    result = review_code("public User getUser(String id) { return repo.findById(id); }")

    assert result.severity == "HIGH"
    assert len(result.issues) == 1
    assert result.issues[0].type == "UNHANDLED_OPTIONAL"

def test_handles_malformed_json(mock_anthropic):
    mock_anthropic.messages.create.return_value = make_mock_response("{broken json")

    with pytest.raises(ValueError, match="Failed to parse"):
        review_code("any code")

def test_wraps_input_in_xml_delimiters(mock_anthropic):
    mock_anthropic.messages.create.return_value = make_mock_response(
        '{"severity": "NONE", "issues": []}'
    )

    review_code("String foo = 'bar';")

    # Assert the user message contained XML delimiters
    call_args = mock_anthropic.messages.create.call_args
    messages = call_args.kwargs["messages"]
    user_content = messages[0]["content"]

    assert "<code_to_review>" in user_content
    assert "</code_to_review>" in user_content
```

### Node.js / TypeScript (Jest)

```typescript
import { jest } from "@jest/globals";
import Anthropic from "@anthropic-ai/sdk";

// Mock the entire SDK module
jest.mock("@anthropic-ai/sdk");

describe("CodeReviewService", () => {
  let mockCreate: jest.MockedFunction<typeof Anthropic.prototype.messages.create>;

  beforeEach(() => {
    const mockClient = new Anthropic() as jest.Mocked<Anthropic>;
    mockCreate = mockClient.messages.create as jest.MockedFunction<typeof mockClient.messages.create>;
  });

  test("should return parsed review result", async () => {
    const mockResponse = {
      content: [{ type: "text", text: '{"severity": "HIGH", "issues": [{"type": "UNHANDLED_OPTIONAL", "description": "test", "recommendation": "fix"}]}' }],
      stop_reason: "end_turn",
    };
    mockCreate.mockResolvedValue(mockResponse as any);

    const result = await reviewCode("public User getUser(String id) { return repo.findById(id); }");

    expect(result.severity).toBe("HIGH");
    expect(result.issues).toHaveLength(1);
  });

  test("should wrap input in XML delimiters", async () => {
    mockCreate.mockResolvedValue({
      content: [{ type: "text", text: '{"severity": "NONE", "issues": []}' }],
      stop_reason: "end_turn",
    } as any);

    await reviewCode("String foo = 'bar';");

    const callArgs = mockCreate.mock.calls[0][0] as Anthropic.MessageCreateParams;
    const userContent = (callArgs.messages[0].content as string);
    expect(userContent).toContain("<code_to_review>");
    expect(userContent).toContain("</code_to_review>");
  });
});
```

---

## Test Organisation

Organise AI unit tests separately from integration tests that make real LLM calls:

```
src/
  test/
    java/
      unit/             ← fast, no LLM calls, run on every commit
        CodeReviewServiceTest.java
        PromptTemplateTest.java
        ResponseParserTest.java
      integration/      ← slow, real LLM calls, run on merge
        CodeReviewIntegrationTest.java
        SemanticAssertionTest.java
      eval/             ← expensive, LLM-as-a-Judge, run nightly or on main
        RelevancyEvaluationTest.java
```

---

## Check Your Understanding

1. A colleague argues that unit tests for AI code are pointless because "you can't test the AI." How would you respond, and what specifically would you test?

2. You need to verify that your RAG service correctly handles the case where no relevant chunks are found. Write the mock setup and assertion for this test case.

3. Your team has a CI pipeline that runs all tests on every pull request. You have added 30 tests that call the real LLM. What is the problem with this arrangement, and how would you fix it?

4. Write a unit test that verifies your system prompt contains the word "MUST NOT" at least once. Why might this be a useful assertion?

5. You discover that your `parseResponse` method silently returns `null` when the model returns an unexpected field name (e.g., `"issues_found"` instead of `"issues"`). Write the test that would have caught this, and describe the fix.
