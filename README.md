# Prompt Engineering Training Programme

**Edition**: 2.1 (7-module, includes real-world roles capstone)\
**Date**: 2026-03-20\
**Audience**: Development teams, DevOps engineers, architects, and technical leads adopting AI-assisted workflows\
**Primary language**: Java / Spring AI. Python and Node.js/TypeScript equivalents throughout Modules 1–6.

---

## How to Use This Programme

**With an AI tutor (recommended):** Open this directory in Claude Code or Claude Desktop. The AI tutor **Alex** will guide you through the content, run you through exercises, and answer questions. See [AGENTS.md](AGENTS.md) for how Alex works.

**Self-paced:** Read the modules directly. Each section is self-contained and ends with check-your-understanding questions. You do not need the AI tutor to benefit from the material.

---

## Prerequisites

- A general technical background (developer, DevOps engineer, architect, or technical lead)
- No prior AI or machine learning experience required
- Module 3 (Programming with AI APIs) requires a development environment with Python, Java, or Node.js

---

## Module Index

Seven modules, ordered by priority. Start at Module 1 regardless of experience level.

| Module | Title | Audience | Duration |
|--------|-------|----------|----------|
| **1** | [Foundations and Prompt Design](modules/module-1-foundations/) | All participants | ~2 hours |
| **2** | [Context, Memory, and RAG Architecture](modules/module-2-context-and-rag/) | Architects, senior developers | ~3 hours |
| **3** | [Programming with AI APIs](modules/module-3-programming-with-ai-apis/) | Developers | ~4 hours |
| **4** | [Testing, Evaluation, and CI/CD](modules/module-4-testing-and-evaluation/) | Developers, DevOps | ~3 hours |
| **5** | [Security and Trust](modules/module-5-security-and-trust/) | All technical participants | ~2.5 hours |
| **6** | [Enterprise Governance and Compliance](modules/module-6-enterprise-governance/) | Leads, architects, management | ~3 hours |
| **7** | [Real-World Roles: Prompt Engineering in a Working Agent Team](modules/module-7-real-world-roles/) | All participants who have completed Module 1 | ~2.5 hours |

**Total**: ~20 hours (3.5-day instructor-led or self-paced over 2–3 weeks)

---

## Module Descriptions

### Module 1 — Foundations and Prompt Design

**Start here.** No prerequisites. The most important module in the programme.

Covers how LLMs process input, why they hallucinate, and how to structure prompts reliably using the four-pillar approach. Includes the persona pattern, XML delimiters, and Chain-of-Thought reasoning. Lab: build a code reviewer in Java, Python, or Node.js.

### Module 2 — Context, Memory, and RAG Architecture

For architects and senior developers building AI features that go beyond single-turn prompts.

Covers context window mechanics, the "Lost in the Middle" problem, memory patterns, embeddings, vector search, and RAG architecture (ingestion, indexing, retrieval). Lab: build a RAG documentation assistant.

### Module 3 — Programming with AI APIs

The core developer content. Build real features with the Anthropic API, Spring AI, and equivalent SDKs.

Covers API fundamentals, five named prompt design patterns, function calling and tool loops, and agentic patterns (ReAct, multi-agent architectures, failure modes and guardrails).

### Module 4 — Testing, Evaluation, and CI/CD

How to know your AI feature is working — and know when it stops.

Covers unit testing AI features, LLM-as-a-Judge evaluation, golden datasets, CI/CD integration (Gradle, pytest, Jest, GitHub Actions), and prompt regression testing. Lab: production hardening and red-team challenge.

### Module 5 — Security and Trust

High-stakes content. Every developer deploying AI features should complete sections 01 and 02 as a minimum.

Covers the Control Plane / Data Plane model, prompt injection (direct and indirect via RAG), defence-in-depth, API key management, PII scrubbing pipelines, and a responsible deployment checklist.

### Module 6 — Enterprise Governance and Compliance

For technical leads, architects, and management involved in AI adoption decisions.

Covers NIST AI RMF and ISO 42001 in plain language, model selection and vendor risk, the EU AI Act and GDPR, LLMOps (prompt drift, model versioning, production monitoring), and change management with ROI measurement.

### Module 7 — Real-World Roles: Prompt Engineering in a Working Agent Team

The applied capstone module. Grounded in a real production multi-agent system — the Podzone Agent Team.

Three roles examined in depth: Infrastructure Operator (GitOps auditing, kubectl access, escalation patterns), Developer/Coder (schema-first testing, cost-aware context management, scope constraints), and Team Lead (decomposition, delegation, backlog orchestration). Section 04 synthesises all three into cross-role collaboration patterns, handoff design, and the failure modes that arise when agent boundaries blur.

**Prerequisite:** Module 1. Recommended: Modules 3 and 4 before section 02.

---

## Learning Paths

### All participants (foundation)

Module 1 → Module 5 → Module 6.01 → Module 7

### Developer track (full)

Module 1 → Module 2 → Module 3 → Module 4 → Module 5 → Module 7.02

### Architect / technical lead track

Module 1 → Module 2 → Module 3.01-02 → Module 5 → Module 6 → Module 7.03-04

### Management / team lead track

Module 1 → Module 6 → Module 5.04 → Module 7.03

### Infrastructure / DevOps track

Module 1 → Module 3.04 → Module 5 → Module 7.01 → Module 7.04

### Fast track (experienced developers)

Module 1.03 → Module 3 → Module 4 → Module 5.02 → Module 7

---

## Code Examples

All code examples in Modules 1–6 are in three languages:

- **Java / Spring AI** — primary; full examples for all topics
- **Python / Anthropic SDK** — parallel examples throughout
- **Node.js / TypeScript / @anthropic-ai/sdk** — parallel examples throughout

Module 7 uses real-world configuration and operational patterns rather than code examples.

---

## Reference Material

The `resources/` directory contains a list of source PDFs used to develop this programme. These are reference documents for instructors and participants who want to go deeper into the underlying curriculum design.

See [resources/source-material.md](resources/source-material.md).

---

## About This Programme

This programme was developed for the Podzone Agent Team's training initiative (PROJ-009). The content in Modules 1–6 is drawn from an industry prompt engineering curriculum. Module 7 is original material synthesised from real operational experience running a multi-agent AI team in production.
