# Module 7 — Real-World Roles: Prompt Engineering in a Working Agent Team

**Audience**: All participants who have completed at least Module 1\
**Prerequisites**: Module 1 (Foundations and Prompt Design). Module 3 and Module 4 provide useful context for sections 02 and 04 but are not required.\
**Duration**: ~2.5 hours

---

## Why This Module Exists

Every other module in this programme teaches you *how* to engineer prompts. This module shows you what that looks like when it is actually working — across three distinct professional roles, in a production multi-agent system.

The Podzone Agent Team is a real operational AI squad: three agent roles (Infrastructure Operator, Developer/Coder, and Team Lead), each running as a separate Claude instance in its own IDE or API context, collaborating through a file-based messaging protocol and a shared Qdrant vector store. The team manages a GitOps platform — Kubernetes clusters, Helm chart deployments, Cloudflare ingress routing, and the GitOpsAPI application that automates all of it.

This module distils real prompt patterns, real failure modes, and real collaboration structures from that team. It is grounded in operational logs, agent persona files, escalation records, and design decisions made under time and cost pressure.

---

## What You Will Learn

1. **Infrastructure Operator perspective** — how a cluster-state agent uses prompts to audit GitOps repos, drive kubectl queries, and escalate decisions it cannot make itself
2. **Developer/Coder perspective** — how an implementation agent uses prompts to manage context across a large codebase, avoid regressions, and operate within strict scope boundaries
3. **Team Lead perspective** — how an orchestrating agent uses prompts for task decomposition, backlog management, and agent delegation without touching implementation
4. **Cross-role collaboration** — how these three agent types hand off work to each other, maintain shared context without re-reading large files, and avoid the failure modes that arise when agent boundaries blur

---

## Sections

| Section | Topic | Estimated time |
|---------|-------|----------------|
| [01](01-infrastructure-operator.md) | The Infrastructure Operator — GitOps auditing, kubectl access, escalation patterns | 35 minutes |
| [02](02-developer-coder.md) | The Developer/Coder — codebase navigation, schema-first testing, cost-aware context | 40 minutes |
| [03](03-team-lead-orchestration.md) | Team Lead Orchestration — decomposition, delegation, backlog management | 35 minutes |
| [04](04-cross-role-collaboration.md) | Cross-Role Collaboration — handoff patterns, shared context, failure modes | 30 minutes |

---

## Key Themes

- **Role boundaries are not just organisational — they are prompt engineering constraints.** Each agent is given a specific persona, a specific scope, and explicit instructions about what it does not do. Blurring these boundaries in a prompt is a reliable way to produce uncontrolled, unauditable behaviour.
- **Context management is the operational skill.** The most expensive failure mode in a multi-agent system is not bad output — it is re-reading the same large files in every session because context was not managed deliberately.
- **Escalation is a first-class output.** A well-designed agent writes a structured escalation document as a valid terminal state. It does not attempt decisions outside its authorisation boundary.
- **The file-based inbox/outbox pattern is a concrete implementation of agent trust boundaries.** Understanding it gives you a mental model you can apply to any inter-agent communication design.

---

## Prerequisites

- [Module 1 — Foundations and Prompt Design](../module-1-foundations/)
- [Module 3 — Programming with AI APIs](../module-3-programming-with-ai-apis/) (recommended for section 02)
- [Module 4 — Testing, Evaluation, and CI/CD](../module-4-testing-and-evaluation/) (recommended for section 02)

---

## Next Step

After completing this module, proceed to:
- **Module 6** (Enterprise Governance) to see how the governance frameworks described there apply to real multi-agent team structures
- **Module 5** (Security and Trust) if you want to think through the injection risks in a file-based inter-agent messaging system
