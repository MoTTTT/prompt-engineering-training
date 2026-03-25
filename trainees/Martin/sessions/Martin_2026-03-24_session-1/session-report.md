# Session Report — Martin — 2026-03-24 — Session 1

## Modules and Sections Covered

This session was applied and architectural rather than module-sequential. Content covered maps to:

- **Module 6.04 (LLMOps / Cost Management):** Token economics, prompt caching mechanics, model tiering by role, offloading strategy for long-context workloads
- **Module 3.04 (Agentic Patterns):** Human:Agent:Project workflow, task repo pattern, single-session vs multi-sprint repo design, fleet vs aggregated agent design
- **Module 2.03 (RAG Architecture):** Qdrant as operational state store, baseline accumulation pattern for time-series context, output-context-index.md pattern
- **Module 5.01/5.02 (Trust Boundaries / Prompt Injection):** Two-tier observability agent design, prompt injection via data pipeline, Telegram channel as untrusted input surface, GitOps-managed thresholds

## Key Concepts Landed

1. Long-context reasoning is the hardest workload to offload to local models — context reduction (workspace scoping) is the primary cost lever, not model substitution
2. Prompt caching: cache write (1.25x), cache read (0.1x) — stable prefixes at top of CLAUDE.md, dynamic task briefs appended after
3. Claude Code loads skills from `.claude/skills/` relative to workspace root — workspace file controls skill set
4. Entire Claude Code config (CLAUDE.md, settings.json, skills) can live in a git repo; state/memory proxied via versioned files
5. `claude.projectInstructions` in `.code-workspace` + CLI `--print` flag enables fully autonomous unattended task runs
6. Observability agent: Tier 1 (scheduled, read-only, processes raw data) separated from Tier 2 (write, triggered, incident management) — prompt injection defence by design
7. Baseline accumulation is not an LLM job — statistical summary pipeline writes to Qdrant, LLM reads summaries for pattern interpretation

## Infrastructure Issues Encountered

- `TRAINING_REVIEW_GIT_TOKEN` not set on this workstation — resolved by Martin clarifying he is the coordinator and this machine can push without it. Token check in CLAUDE.md is appropriate for trainee machines but needs a bypass mechanism for coordinator sessions.
- Workspace file path bug: `.code-workspace` files with relative paths only work reliably when placed at repo root with `"path": "."`. Bug confirmed in generated workspace files.
- Session skills (`session-start`, `session-end`) not available in this workspace — skills live in `podzoneAgentTeam/.claude/skills/` and don't travel to other workspaces unless installed by workspace init.

## Trainee Feedback (verbatim)

> **Ease of use:** "Ease of use is very good (the course content, the workstation setup we have discussed)"
>
> **Most useful learning:** "I learned how to better structure working sessions, and how to manage the use of internal infrastructure based on role/task, and how to securely separate the agents that are exposed, from the agents that need to be trusted at a higher level."
>
> **What worked well:** "The conversational tone is a winner. The Socratic approach is working well it seems."
>
> **Difficult or unnecessary:** "Structuring my prompts requires careful focus, but that is necessary."
>
> **Other suggestions:** "No other suggestions, thanks."

## Recommendations for Next Session

- Pick up at the open question: trust boundary design for the observability agent using Module 5 framework as formal validation pass
- Cover Module 5.01 and 5.02 in depth — Martin has a live use case that maps directly
- Continue into Module 3.03 (function calling and tools / MCP) — the MCP consolidation question for the observability fleet was left open
- Martin is also a Cycle 1 reviewer (C1-T001b) — collect module-level improvement feedback as content is covered
