# Module 4 — Testing and Evaluation

## Overview

Testing AI features requires a different mindset from testing traditional software. The output is non-deterministic, so `assertEquals` tests are brittle. The quality of the output is semantic, not syntactic — you need to ask "does this response mean the right thing?" not "does this response match this exact string?" This module covers the full testing stack for AI systems: from mocking LLM calls in unit tests, to semantic evaluation, to CI/CD integration and prompt regression testing.

---

## Sections

| Section | Title | Estimated time |
|---------|-------|---------------|
| [01](01-unit-testing-ai-features.md) | Unit Testing AI Features | 40 minutes |
| [02](02-semantic-evaluation.md) | Semantic Evaluation | 50 minutes |
| [03](03-cicd-integration.md) | CI/CD Integration | 40 minutes |
| [04](04-prompt-regression-testing.md) | Prompt Regression Testing | 40 minutes |
| [Lab 03](lab-03-production-hardening.md) | Production Hardening | 90 minutes |

---

## Learning Objectives

By the end of this module, you will be able to:

- Write unit tests for AI features by mocking LLM responses
- Implement semantic assertions and the LLM-as-a-Judge pattern
- Integrate AI tests into a CI/CD pipeline with appropriate cost management
- Set up a prompt regression test suite using Promptfoo
- Run the red-team challenge: attack your own system and convert findings into regression tests

---

## Prerequisites

- Module 3 (Programming with AI APIs)
- Familiarity with your team's test framework (JUnit 5, pytest, or Jest)
- Understanding of your CI/CD pipeline

---

## The Three Levels of AI Testing

The module covers testing at three levels of increasing cost and semantic richness:

| Level | What it tests | Cost | When to run |
|-------|--------------|------|------------|
| **Unit tests** | Prompt construction, JSON parsing, error handling | No LLM calls | On every commit |
| **Semantic assertions** | That the real LLM returns appropriate content | One LLM call per test | On every PR merge |
| **LLM-as-a-Judge** | Whether the response is relevant, faithful, and correct | Two LLM calls per test | On merge to main; nightly |
