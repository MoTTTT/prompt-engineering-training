# 4.03 — CI/CD Integration

## Key Insight

Integrating AI tests into CI/CD requires explicitly managing the trade-off between test coverage and cost. Every LLM call in a test pipeline has a real monetary cost and latency overhead. The solution is a tiered test strategy: fast, cheap tests run on every commit; slow, expensive tests run on merge to main or nightly. Failing to make this distinction either runs up large bills or gives developers no feedback on AI quality.

---

## The Tiered Test Strategy

| Tier | What runs | When | Cost per run |
|------|---------|------|------------|
| **Unit** | Mocked LLM calls; prompt template checks; JSON parsing | Every commit | ~$0 |
| **Semantic assertions** | Real LLM; string-level quality checks | Every PR merge | ~$0.01–0.05 |
| **LLM-as-a-Judge** | Real LLM; semantic quality; golden dataset | Merge to main; nightly | ~$0.10–1.00 |

This tiering is the same principle as the fast/slow test split in traditional software: keep the fast feedback loop cheap and immediate, run expensive evaluation less frequently.

---

## Java: Gradle + JUnit 5 + Test Tags

Use JUnit 5 tags to separate test tiers:

```java
// Unit test — no tags needed, runs always
class PromptTemplateTest { ... }

// Integration test — real LLM, runs on PR merge
@Tag("integration")
class SemanticAssertionTest { ... }

// Evaluation test — expensive, runs on main or nightly
@Tag("eval")
class LlmJudgeEvaluationTest { ... }
```

Configure Gradle to include/exclude tags:

```groovy
// build.gradle

test {
    useJUnitPlatform {
        // By default: run everything except eval
        excludeTags("eval")
    }
}

// New task: run only integration tests (for PR merge CI step)
tasks.register("integrationTest", Test) {
    description = "Run integration tests against real LLM"
    group = "verification"

    useJUnitPlatform {
        includeTags("integration")
        excludeTags("eval")
    }

    // Environment variable for LLM API key
    environment("ANTHROPIC_API_KEY", System.getenv("ANTHROPIC_API_KEY"))

    // Longer timeout for LLM calls
    timeout = Duration.ofMinutes(5)
}

// New task: run full eval suite (for main merge / nightly)
tasks.register("evalTest", Test) {
    description = "Run LLM-as-a-Judge evaluation suite"
    group = "verification"

    useJUnitPlatform {
        includeTags("eval")
    }

    environment("ANTHROPIC_API_KEY", System.getenv("ANTHROPIC_API_KEY"))
    timeout = Duration.ofMinutes(30)
}

// Wire into the check task: unit + integration, but not eval
tasks.named("check") {
    dependsOn("test", "integrationTest")
}
```

---

## GitHub Actions Workflow

### Java / Gradle

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: "21"
          distribution: "temurin"
      - name: Run unit tests
        run: ./gradlew test
        # No LLM API key needed — mocked tests only

  integration-tests:
    name: Integration Tests (LLM)
    runs-on: ubuntu-latest
    if: github.event_name == 'push' || github.event.pull_request.base.ref == 'main'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: "21"
          distribution: "temurin"
      - name: Run integration tests
        run: ./gradlew integrationTest
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        timeout-minutes: 10

  eval-tests:
    name: Evaluation Suite (Nightly)
    runs-on: ubuntu-latest
    if: github.event_name == 'schedule' || github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: "21"
          distribution: "temurin"
      - name: Run evaluation suite
        run: ./gradlew evalTest
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY_EVAL }}  # separate key with spending limit

# Nightly schedule
on:
  schedule:
    - cron: "0 2 * * *"  # 02:00 UTC daily
```

**Note the separate API key for eval (`ANTHROPIC_API_KEY_EVAL`):** Use a separate API key with a spending limit for CI evaluation. If a test suite accidentally generates too many tokens (e.g., an infinite retry loop), the spending limit caps the damage.

### Python (pytest)

```python
# conftest.py — pytest configuration
import pytest

def pytest_configure(config):
    config.addinivalue_line("markers", "unit: fast unit tests, no LLM calls")
    config.addinivalue_line("markers", "integration: integration tests, real LLM calls")
    config.addinivalue_line("markers", "eval: evaluation tests, expensive LLM-as-a-Judge")

# Run markers as arguments:
# pytest -m "unit" — unit tests only
# pytest -m "integration" — integration tests
# pytest -m "eval" — evaluation suite
```

```yaml
# .github/workflows/ci.yml (Python)
- name: Run unit tests
  run: pytest -m "unit" -v

- name: Run integration tests
  run: pytest -m "integration" -v
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

### Node.js (Jest)

```javascript
// jest.config.js
module.exports = {
  projects: [
    {
      displayName: "unit",
      testMatch: ["**/*.unit.test.ts"],
      // No LLM API key required
    },
    {
      displayName: "integration",
      testMatch: ["**/*.integration.test.ts"],
      globals: {
        ANTHROPIC_API_KEY: process.env.ANTHROPIC_API_KEY,
      },
    },
  ],
};
```

```bash
# Run only unit tests
npx jest --selectProjects unit

# Run integration tests
npx jest --selectProjects integration
```

---

## Cost Management for CI Evaluation

### Set API spending limits

Most LLM providers allow you to set monthly spending limits per API key. Set limits on CI keys to prevent runaway costs.

### Cache LLM responses in tests

For stable tests with identical inputs, cache the LLM response:

```java
// Spring AI supports caching via the prompt caching feature
// For test data that never changes, consider storing expected responses
// as fixtures and returning them directly without calling the LLM

// Alternatively, use a recording/replay approach for integration tests:
// 1. Run integration tests once against the real LLM, capture responses
// 2. Store captured responses as fixtures
// 3. Replay fixtures in subsequent runs unless explicitly reset
```

### Monitor CI costs

Add a step to report token usage after eval runs:

```yaml
- name: Report evaluation costs
  if: always()
  run: |
    echo "Evaluation run completed. Check LLM provider dashboard for usage:"
    echo "https://console.anthropic.com/usage"
```

---

## Gate Conditions: What Failing Means

Define clearly what failing an AI test means for the pipeline:

| Test type | Failure means | Action |
|-----------|--------------|--------|
| Unit test | Broken prompt plumbing | Block merge — hard failure |
| Semantic assertion | Quality regression | Block merge — hard failure |
| LLM-as-a-Judge (main) | Quality threshold not met | Block merge — hard failure |
| LLM-as-a-Judge (nightly) | Nightly quality check failed | Alert engineering team, review next day |

For the nightly evaluation suite, you may choose to alert rather than block, since the failure may be due to model provider variance rather than a code regression.

---

## Check Your Understanding

1. Your team runs all LLM tests on every pull request. The test suite takes 8 minutes and costs $0.50 per run. With 40 pull requests per week, what is the monthly CI cost for LLM tests, and what would you change?

2. A developer adds a LLM-as-a-Judge test to the unit test suite (the suite that runs on every commit). What is the problem with this, and how would you enforce the test tier separation?

3. A nightly evaluation run shows that the "faithfulness" score dropped from 95% to 78% after a Kubernetes deployment on Monday. The code has not changed. What would you investigate?

4. Write the GitHub Actions step that runs integration tests only on pull requests targeting `main`, not on pull requests to feature branches.

5. Your evaluation test suite uses the same API key as your production service. A runaway evaluation loop consumes $500 of credits. What architectural change would prevent this?
