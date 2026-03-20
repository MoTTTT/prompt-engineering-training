# 7.04 — Cross-Role Collaboration: Handoff Patterns, Shared Context, and Failure Modes

## Key Insight

A multi-agent team is not just multiple AI instances running in parallel — it is an information routing system. Every inter-agent communication is a prompt, a context transfer, and a trust boundary. The quality of collaboration between roles depends almost entirely on how well these three concerns are designed: what information is passed, in what format, and whether the receiving agent can trust the sender's output without re-verifying everything from scratch.

---

## The Communication Architecture

The Podzone Agent Team uses a file-based inbox/outbox pattern:

```
Team Lead                    Developer/Coder          Cluster Operator
     |                              |                        |
     | --> agents/claude-code/      |                        |
     |     incoming/YYYY-MM-DD-*.md |                        |
     |                              |                        |
     | --> agents/cluster-operator/ |                        |
     |     incoming/YYYY-MM-DD-*.md |                        |
     |                              |                        |
     |                              | --> agents/team-lead/  |
     |                              |     incoming/*.md      |
     |                              |                        |
     |                              |                        | --> agents/team-lead/
     |                              |                              incoming/*.md
     |                              |                              (escalations)
     |                              |
     |                              | --> agents/cluster-operator/
     |                              |     incoming/*.md (task requests)
```

Each agent reads from its own `incoming/` directory and writes to `outgoing/` plus the target agent's `incoming/`. The Team Lead reads from `agents/team-lead/incoming/` (escalations and questions from any agent) and writes to other agents' `incoming/` (task assignments).

This file-based model has a significant property: **messages are durable and inspectable.** A human (Martin) can read the full message history at any time, in any order. An agent that arrives mid-project can read the full incoming queue to reconstruct context. There is no lost-in-transit problem.

---

## The Three Handoff Patterns

### Pattern 1: Team Lead → Specialist Agent (Task Assignment)

**Trigger:** Team Lead receives high-level direction or an unblocked task from the backlog.

**Output format:** Structured task file in `agents/{specialist}/incoming/YYYY-MM-DD-{topic}.md`

**What makes this handoff work:**
- Success criteria are specific and checkable — the receiving agent knows what "done" looks like
- Context is pre-assembled — the receiving agent does not need to re-read large spec files
- Dependencies are stated — the agent knows what must be true before starting
- Authority scope is explicit — the agent knows what it is authorised to do and what requires escalation

**What makes this handoff fail:**
- Vague task description — the agent makes assumptions that may not match the intent
- Missing context — the agent reads large files to fill the gap, consuming context budget
- Undefined success criteria — the agent produces something that "might be right" and cannot verify

**Example:** The task assignment from Team Lead to Cluster Operator for the cluster workload audit (PROJ-007/T-001) specified exactly what to produce (an audit report with app × cluster matrix), where to put it (`agents/cluster-operator/outgoing/`), and what it was blocking (T-002 and T-003 could not start without it). The Cluster Operator completed the task and sent a completion notice within one session.

### Pattern 2: Specialist Agent → Team Lead (Escalation or Completion Notice)

**Trigger A — Escalation:** Agent encounters a decision it is not authorised to make.

**Output format:** Escalation document in `agents/team-lead/incoming/YYYY-MM-DD-{topic}.md`

**What makes an escalation work:**
- The decision question is stated precisely — not "what should I do?" but specific options with implications
- The impact is stated — what is blocked until the decision is made, and what is not blocked
- The options are labelled (A, B, C) with trade-offs for each

**What makes an escalation fail:**
- Vague question ("I'm not sure about the security approach") — the Team Lead cannot make a decision without specific options
- Missing impact statement — the Team Lead cannot prioritise the escalation
- Agent attempts to make the decision anyway — produces an undocumented architectural choice

**Example:** The Cluster Operator escalated three decisions blocking the Apache-to-Cloudflare migration. Each decision was formatted as: issue description, risk, named options with implications, explicit request. The Team Lead (Martin) responded within one session with Option B (API key auth for Ollama), Harbor decommission, and hostname exclusions. All three unblocked the same working day.

**Trigger B — Completion Notice:** Agent completes a task and the downstream work needs to be unblocked.

**Output format:** Completion document in `agents/team-lead/incoming/` or `agents/claude-code/incoming/` depending on who needs to pick up next.

**What makes a completion notice work:**
- Status is explicit (complete, partially complete, blocked)
- Outstanding items carried forward are listed
- Next agent's task is identified

### Pattern 3: Cluster Operator → Developer/Coder (Direct Task Request)

**Trigger:** Cluster Operator identifies infrastructure work that requires GitOpsAPI feature development.

**Output format:** Task request in `agents/claude-code/incoming/YYYY-MM-DD-{topic}.md`

**What makes this handoff work:**
- The Cluster Operator's findings are the source of truth — the Developer/Coder trusts them without re-querying the cluster
- The required feature is described in terms of the GitOpsAPI surface (endpoint, request body, expected behaviour) — not as an infrastructure configuration description
- The Cluster Operator's authority boundary is respected — the task request asks for a GitOpsAPI feature, not for the Developer/Coder to write GitOps manifests

**What makes this handoff fail:**
- The Cluster Operator describes what the cluster needs rather than what the API should do — the Developer/Coder cannot act on "the cluster needs a Gateway" without knowing what API endpoint to implement
- The task request describes both the API feature and the manifest work — scope confusion, two agents may both attempt the manifest

---

## Shared Context: The Qdrant Layer

All three roles have access to the shared Qdrant vector store (`http://localhost:6333`), which holds:

- `gitopsgui-specs` — GitOpsAPI requirements, architecture, task breakdowns
- `planning-docs` — Tasks, decisions, strategic documents from the Agent Team

The shared context layer allows agents to work from the same source of truth without each agent loading the same large files into context. This is the technical implementation of "don't re-read large files" — the documents are indexed once and queried many times.

### How agents use Qdrant differently

The **Developer/Coder** queries Qdrant first for every task, before reading any local files:
```python
client = QdrantClient(url="http://localhost:6333")
results = client.search(
    collection_name="gitopsgui-specs",
    query_text="ClusterSpec fields and constraints",
    limit=3
)
```

The **Team Lead** loads context *into* Qdrant when assembling complex tasks:
- Creates a document with the relevant spec excerpt
- Includes the document reference in the task file
- The receiving agent can query Qdrant instead of receiving a large inline context block

The **Cluster Operator** contributes to Qdrant indirectly — its audit reports and findings are valuable inputs, but the pre-assembly step (loading them to Qdrant) is a Team Lead responsibility.

### The pre-assembly vs self-assembly decision

In the Podzone team, the CW-004 decision resolved this as **pre-assembly**: the Team Lead assembles context and includes it in task files or Qdrant references. Agents do not query Qdrant speculatively — they only query when the task file explicitly references a collection query.

This trades Team Lead overhead for agent reliability: the Team Lead session costs more per task creation, but the receiving agents produce better first-pass outputs.

---

## Trust and Verification in the File-Based System

The file-based inbox/outbox model creates an implicit trust model: each agent trusts the content of its `incoming/` directory without re-verifying from first principles. This is both a feature and a risk.

**Feature:** The Developer/Coder trusts the Cluster Operator's audit findings without re-querying the cluster. This is efficient and appropriate — the Cluster Operator has kubectl access; the Developer/Coder does not.

**Risk:** If the incoming queue is corrupted, out of order, or contains instructions from an unexpected source, the agent may act on bad instructions. This is the multi-agent equivalent of indirect prompt injection (see Module 5.02) — a message in the inbox that does not originate from the expected sender.

**Mitigations in the Podzone design:**
- Inboxes are managed directories in a git repository — changes are versioned and auditable
- The file naming convention (`YYYY-MM-DD-{sender}-{topic}.md`) makes the source visible
- Agents check their own AGENTS.md for scope before acting on any incoming message
- Escalation is the response when instructions exceed authorised scope

### The injection risk at agent boundaries

A developer agent that accepts GitOps manifest instructions from a file in its `incoming/` directory is potentially vulnerable to instructions that appear to come from the Cluster Operator but do not. In the Podzone system this is mitigated by git history (human reviewer can see what files were written and by whom), but in an automated pipeline this trust boundary would need cryptographic authentication.

This is an area where the real-world system acknowledges a gap: the file-based messaging protocol is a pragmatic starting point, not a production-grade trust model.

---

## Session Boundaries and Context Handoff

The most challenging collaboration problem in a multi-agent team is the session boundary: each new agent session starts with no memory of previous sessions. For a team to function across multiple work sessions, it needs a structured answer to: *how does an agent re-establish context when it starts a new session?*

The Podzone team uses three mechanisms:

### 1. The Audit Trail as Session Handoff

Each agent maintains an `audit-trail.md` that records: date, session actions, decisions made, outstanding items, and next session focus. The audit trail is the first thing read at the start of each new session. For the Cluster Operator:

```
## 2026-03-18 — Session: Internal DNS + Manifests Serving Design
...
Outstanding Cluster Operator tasks (next session):
- Write cluster refactoring roadmap — PROJ-007/T-001 deliverable
- T-002 (CC-066): Deploy Gateway API to gitopsdev — Ready
- T-003 (CC-067): Deploy cloudflared to management cluster — Ready
```

The agent that starts a new session two days later reads this and knows exactly what to pick up. Without the audit trail, it would need to re-read the entire task history to reconstruct this state.

### 2. The AGENTS.md as Persistent Session Context

The AGENTS.md (or CLAUDE.md) for each agent is read at session start and provides: role, scope, capabilities, known constraints, and workflow. This document evolves as the agent's role evolves — it is a living description of the agent's context, not a static persona prompt.

Changes to AGENTS.md are significant events. When the Cluster Operator gained `kubectl` access (a new capability), AGENTS.md was updated with the kubeconfig table and refresh instructions. Any future session that reads AGENTS.md has correct access information without needing to ask.

### 3. The Incoming Queue as Current Work

At every session start, the agent reads all files in its `incoming/` directory. Unprocessed incoming files are carried forward (noted in the audit trail as "unprocessed incoming"). This ensures that no task instruction is silently lost between sessions.

---

## The Collaboration Diagram

```
Martin (Human)
    |
    | high-level direction, decisions
    v
Team Lead
    |
    |--- task assignments ---------> Developer/Coder
    |                                      |
    |--- task assignments ---------> Cluster Operator
    |                                      |
    |<-- escalations (decisions) ----------+
    |<-- completion notices ---------------+
    |<-- escalations (questions) --- Developer/Coder
    |
    |--- decisions, task unblocks -> Cluster Operator
    |--- decisions, task unblocks -> Developer/Coder

Cluster Operator
    |
    +--- task requests (API features) --> Developer/Coder

Qdrant (shared context layer)
    |
    +--- read by Developer/Coder (specs, requirements)
    +--- seeded by Team Lead (task context, decisions)
```

---

## Key Principles for Cross-Role Collaboration Prompt Design

| Principle | Why it matters |
|-----------|---------------|
| Every handoff is a complete context transfer | The receiving agent cannot ask follow-up questions without a round-trip |
| Structured message formats are contracts | Deviating from the format breaks the receiving agent's parsing |
| Trust is scoped to inbox origin | Instructions from unexpected sources should trigger escalation, not action |
| Session boundaries require explicit handoff documentation | No audit trail = no continuity across sessions |
| Shared context should be indexed, not repeated | Qdrant is for content that multiple agents need; inline context is for task-specific details |
| Escalation is the right response to out-of-scope instructions | Acting outside scope under ambiguous instructions is the failure mode |

---

## Check Your Understanding

1. Draw the information flow for this scenario: the Cluster Operator discovers that the GitOpsAPI is missing an endpoint needed to deploy a new application. Who does it send a message to, in what format, and who eventually writes the code?

2. A Developer/Coder session opens and finds three files in its incoming queue from three different senders (Team Lead, Cluster Operator, and an unknown source). How should the agent treat each file, and what should it do with the unknown-source file?

3. The Cluster Operator completes a task but the Team Lead is not running in the current session. What should the Cluster Operator write, where should it write it, and how will the Team Lead find it when it next runs?

4. You are designing a multi-agent system for a software engineering team. Define the inbox/outbox communication pattern for three roles: Architect, Developer, and QA Engineer. What are the trust boundaries, and what happens when a message crosses a boundary it should not?

5. An agent audit trail has not been updated for five sessions. A new session of that agent begins. What information is the agent likely to lack, what is the risk, and how would you recover the current state?
