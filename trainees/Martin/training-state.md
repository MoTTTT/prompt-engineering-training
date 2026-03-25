# Training State — Martin

**Last session:** 2026-03-24 (session 1)

## Completed sections

- Module 6.04 — LLMOps / cost management (applied: token economics, prompt caching, model tiering, offloading strategy)
- Module 3.04 — Agentic patterns (applied: Human:Agent:Project workflow, task repo pattern, fleet vs aggregated agent design)
- Module 2.03 — RAG architecture (applied: Qdrant as operational state store, baseline accumulation pattern)
- Module 5.01/5.02 — Trust boundaries / prompt injection (applied: two-tier observability agent, data pipeline injection, untrusted input surfaces)

## Current position

Next session: formal trust boundary design for the observability agent using Module 5 framework. Then Module 3.03 (function calling / MCP) for the observability fleet consolidation question.

## Open question (carry forward)

Trust boundary design for observability agent fleet — Tier 1 (scheduled/read) vs Tier 2 (triggered/write). MCP tool consolidation to reduce running processes. Martin to confirm incident management platform identity.

## Notes

- Experienced platform architect — all explanations at architect level, no basics needed
- Has Ollama, Qdrant, MCP on-prem. GPU on roadmap.
- Primary pain addressed this session: cost management, workspace structure, agent separation
- Observability agent product is the primary applied use case driving module content
- Also acting as Cycle 1 reviewer (C1-T001b) — collect improvement recommendations throughout
- Spring Java background; DevSecOps / GitOps / Kubernetes context
- Conversational Socratic tone confirmed working well
- Prompt structuring is a conscious effort for Martin — worth noting as a recurring coaching point
