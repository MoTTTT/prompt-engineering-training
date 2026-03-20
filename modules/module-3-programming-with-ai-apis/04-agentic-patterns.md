# 3.04 — Agentic Patterns

## Key Insight

An agent is an LLM given tools and autonomy to use them across multiple steps. The shift from a simple LLM call to an agentic system is a shift from "model generates text" to "model makes decisions." This is powerful and potentially risky. Agents can complete complex multi-step tasks, but they can also loop, hallucinate actions, or be manipulated by injected instructions in their environment. This section covers the architectural patterns you need to use agents effectively — and the guardrails you must build in before deploying them.

---

## The ReAct Pattern

**ReAct** (Reason + Act) is the foundational pattern for agentic behaviour. The model alternates between:

1. **Reasoning:** "What do I know? What do I need to find out? Which tool should I use next?"
2. **Acting:** Calling a tool and observing the result
3. **Reasoning again:** "What did I learn? What do I need to do next?"

This is the loop we implemented in section 3.03. The key architectural element is that the model's reasoning is explicit — it appears in the model's response before the tool call, making it visible and auditable.

In practice with the Anthropic API, the model's reasoning appears in the text content blocks preceding the `tool_use` block. In Chain-of-Thought prompts, you can make this more explicit by instructing the model to write its reasoning in `<thinking>` tags before declaring a tool call.

### When ReAct Works Well

- The task requires information that must be fetched dynamically (database queries, API calls, real-time data)
- The task can be decomposed into a clear sequence of steps
- Each step has a clear success/failure signal
- The set of possible actions is bounded and well-defined

### When to Choose a Simpler Approach

Before building an agent, ask: can this be done with a single LLM call plus a RAG query? Agents add complexity, latency, and failure modes. A well-designed RAG assistant handles most Q&A use cases without a tool loop. Reserve agentic patterns for tasks where the sequence of steps is genuinely dynamic and cannot be pre-specified.

---

## Guardrails for Agents

Agents require guardrails that single-call LLM integrations do not. These are not optional.

### 1. Maximum Step Count

Every tool loop must have a hard limit on the number of steps. Without it, a model that cannot complete a task will loop indefinitely.

```python
MAX_AGENT_STEPS = 10

def run_agent(question: str) -> str:
    messages = [{"role": "user", "content": question}]

    for step in range(MAX_AGENT_STEPS):
        response = call_llm(messages)
        if response.stop_reason == "end_turn":
            return get_text(response)
        if response.stop_reason == "tool_use":
            messages = append_tool_results(messages, response)
            continue

    # If we reach here, the agent did not converge
    raise AgentTimeoutError(f"Agent did not complete within {MAX_AGENT_STEPS} steps")
```

### 2. Tool Input Validation

Validate every tool call argument before executing. Return structured error messages (not exceptions) so the model can self-correct:

```python
def execute_tool_safely(tool_name: str, tool_input: dict) -> dict:
    try:
        validate_tool_input(tool_name, tool_input)
        return TOOL_REGISTRY[tool_name](**tool_input)
    except ValidationError as e:
        return {"error": f"Invalid parameters: {e}", "hint": "Check the tool definition for required fields"}
    except Exception as e:
        return {"error": f"Tool execution failed: {type(e).__name__}", "details": str(e)}
```

### 3. Confirmation for Destructive Actions

For actions that are irreversible (deleting records, sending emails, modifying production configuration), require explicit human confirmation before execution:

```java
@Tool(description = "Deploys a new version to the staging environment. Requires human confirmation.")
public DeployResult deployToStaging(String serviceName, String version) {
    // Check if this action has been confirmed by a human in this session
    if (!confirmationService.isConfirmed("deploy-staging-" + serviceName)) {
        return DeployResult.requiresConfirmation(
            "Please confirm: deploy " + serviceName + " version " + version + " to staging?",
            "deploy-staging-" + serviceName
        );
    }
    return deploymentService.deploy(serviceName, version, "staging");
}
```

### 4. Scope Constraints in the System Prompt

Explicitly define what the agent is and is not permitted to do:

```
You are a DevOps assistant with access to tools for querying cluster status and
viewing deployment history.

You MUST NOT:
- Trigger deployments without explicit user confirmation
- Modify production configuration
- Access credentials or secrets
- Execute arbitrary shell commands

If a user asks you to do something outside this scope, say:
"That action is outside my permitted scope. Please contact the DevOps team directly."
```

---

## Multi-Agent Architectures

For complex workflows, a single agent with many tools becomes difficult to control. Multi-agent architectures delegate sub-tasks to specialised agents:

```
User request
    |
    v
[Orchestrator Agent]
    - Understands the high-level task
    - Decomposes it into sub-tasks
    - Dispatches to appropriate sub-agents
    |
    +---> [Documentation Search Agent] — read-only access to doc store
    |
    +---> [Code Review Agent] — analyses code, no execution rights
    |
    +---> [Deployment Agent] — limited to staging, requires confirmation
    |
    v
Orchestrator synthesises results and returns to user
```

**Why this is better than one agent with many tools:**

- Each sub-agent has a narrow scope and limited permissions
- Sub-agents can be independently tested and replaced
- The orchestrator's reasoning is simpler when it delegates to specialists
- A compromised sub-agent has limited blast radius

**Trust in multi-agent architectures:** A sub-agent should not trust instructions from the orchestrator unconditionally. It should validate that the requested action falls within its permitted scope, regardless of where the instruction comes from. An orchestrator that has been compromised via injection should not be able to instruct a deployment sub-agent to deploy to production.

---

## The Agentic Pattern in Practice

Here is a complete minimal orchestrator that dispatches to two sub-agents:

### Python (Anthropic SDK)

```python
import anthropic
import json

client = anthropic.Anthropic()

# Sub-agent 1: Documentation lookup (read-only)
def documentation_agent(query: str) -> str:
    """Answers questions from the documentation corpus."""
    response = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=512,
        system="You are a documentation assistant. Answer questions based on the provided context only. Context: [documentation here]",
        messages=[{"role": "user", "content": query}]
    )
    return response.content[0].text

# Sub-agent 2: Status checker (read-only database access)
def status_agent(cluster_name: str) -> dict:
    """Returns cluster status from the database."""
    # In real code, query the database
    return {"cluster": cluster_name, "status": "HEALTHY", "nodes": 3}

# Orchestrator tools
ORCHESTRATOR_TOOLS = [
    {
        "name": "ask_documentation",
        "description": "Search the documentation for answers to procedural questions.",
        "input_schema": {
            "type": "object",
            "properties": {"query": {"type": "string"}},
            "required": ["query"]
        }
    },
    {
        "name": "check_cluster_status",
        "description": "Get the current health status of a named cluster.",
        "input_schema": {
            "type": "object",
            "properties": {"cluster_name": {"type": "string"}},
            "required": ["cluster_name"]
        }
    }
]

def run_orchestrator(user_question: str, max_steps: int = 5) -> str:
    messages = [{"role": "user", "content": user_question}]

    for step in range(max_steps):
        response = client.messages.create(
            model="claude-haiku-4-5",
            max_tokens=1024,
            system="You are a DevOps assistant. Use tools to answer questions about clusters and documentation.",
            tools=ORCHESTRATOR_TOOLS,
            messages=messages
        )

        if response.stop_reason == "end_turn":
            return next(b.text for b in response.content if hasattr(b, "text"))

        if response.stop_reason == "tool_use":
            messages.append({"role": "assistant", "content": response.content})
            tool_results = []

            for block in response.content:
                if block.type == "tool_use":
                    if block.name == "ask_documentation":
                        result = documentation_agent(block.input["query"])
                    elif block.name == "check_cluster_status":
                        result = status_agent(block.input["cluster_name"])
                    else:
                        result = {"error": f"Unknown tool: {block.name}"}

                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": json.dumps(result) if isinstance(result, dict) else result
                    })

            messages.append({"role": "user", "content": tool_results})

    raise RuntimeError(f"Orchestrator did not complete within {max_steps} steps")
```

---

## Agent Failure Modes Summary

| Failure mode | Cause | Prevention |
|-------------|-------|-----------|
| Infinite loop | No convergence; error not handled | Hard step limit; structured error returns from tools |
| Hallucinated arguments | Model invents parameter values | Input validation in tool functions; return valid options in errors |
| Cascading errors | Failure in step N causes nonsense in step N+1 | Explicit error states in tool outputs; never return None |
| Scope creep | Model uses tools outside their intended purpose | System prompt constraints; tool descriptions that specify when NOT to use them |
| Injection via environment | Malicious content in retrieved documents overrides instructions | See Module 5: indirect injection defence |

---

## Check Your Understanding

1. What is the difference between a single LLM call with a well-crafted prompt and an agentic loop? Name two specific scenarios where you would choose each.

2. Your agent is meant to complete a task in 3–5 steps but sometimes runs for 30+ steps before stopping. What architectural safeguard would you add, and what debugging information would you log?

3. In a multi-agent architecture, the orchestrator instructs a deployment sub-agent to deploy to production with a parameter the sub-agent does not recognise. How should the sub-agent respond, and why?

4. A user reports that your documentation agent sometimes changes its persona mid-conversation and starts answering questions outside its defined scope. What prompt engineering change and what technical safeguard would you add?

5. When would you choose to manage the tool loop explicitly (as in the Python example) versus using a framework that handles it automatically (as Spring AI does)? What are the trade-offs?
