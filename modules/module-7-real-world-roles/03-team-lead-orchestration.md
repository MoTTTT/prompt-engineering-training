# 7.03 — Team Lead Orchestration: Decomposition, Delegation, and Backlog Management

## Key Insight

An orchestrating agent that cannot touch code, manifests, or infrastructure has a narrowly defined output: structured work instructions delivered to the right agent at the right time. The prompt engineering challenge for this role is not intelligence — it is discipline. The Team Lead must decompose vague high-level direction into precise, actionable tasks; maintain a consistent backlog structure across sessions with no persistent memory; and resist the temptation to solve problems directly rather than routing them.

---

## What the Role Looks Like in Practice

The Team Lead in the Podzone Agent Team runs as an Anthropic API instance (`openclaw` cluster) with Telegram and SSH access for human-in-the-loop interaction. Unlike the Developer/Coder (IDE subscription, zero per-token cost) and the Cluster Operator (IDE subscription), the Team Lead has a real API cost per session. This shapes its operating model significantly.

Its scope is precisely:

- High-level task breakdown (Program → Project → Task → Subtask decomposition)
- Backlog management (`planning/team-tasklist.md`, `planning/tasks.md`)
- Incoming and outgoing documentation management
- Shared context establishment in Qdrant (`planning-docs` collection)
- Agent assignment and READMEFIRST document generation for each active context

What it explicitly is not:

- An infrastructure operator
- A coder
- An architect
- An HR agent (separate role)

These exclusions are not just descriptive — they are operational constraints. When a task arrives that could be handled directly (e.g. the Team Lead could in principle read a GitOps manifest to answer a question), the right response is to route it to the Cluster Operator rather than answer it, because the Team Lead authorising itself to do adjacent work is how role boundaries collapse.

---

## How the Agent Uses AI in Daily Work

### Pattern 1: The 4-Level Work Hierarchy as a Decomposition Framework

The Team Lead operates within a formal work hierarchy:

```
Program (PRG-NNN) — Long-term initiative
  └─ Project (PROJ-NNN) — Stable, repo-aligned work stream
        └─ Task (TASK-NNN or CC-NNN) — Actionable work item
              └─ Subtask — Task breakdown
```

When high-level direction arrives (e.g. "migrate ingress from Apache to Cloudflare Tunnel"), the Team Lead does not pass the whole direction to an agent. It decomposes it:

- **PROJ-007** — Ingress Refactoring (project)
  - **T-001** (CC-065) — Cluster workload audit → Cluster Operator
  - **T-002** (CC-066) — Deploy Gateway API to gitopsdev → Cluster Operator
  - **T-003** (CC-067) — Deploy cloudflared to management cluster → Cluster Operator
  - **T-007** (TASK-060) — iptables automation → Cluster Operator, then reassessed as already complete

Each task has a clear owner, a clear deliverable, and clear dependencies. The decomposition creates a work graph that the Team Lead can use to sequence task delegation.

The prompt engineering lesson: **decomposition is not summarisation.** A vague direction summarised as a task is still vague. A vague direction decomposed into numbered, owned, sequenced tasks with explicit dependencies is actionable.

### Pattern 2: Task Delegation as Structured Document Creation

Every task delegated to another agent is a structured markdown document placed in that agent's `incoming/` directory:

```markdown
# {Task Title} — YYYY-MM-DD HH:MM GMT

**From:** Team Lead
**To:** {Agent Role}
**Priority:** High | Normal | Low
**Blocking:** {Task IDs if this blocks other work}
**Detail:** {Link to detail file if complex}

---

## Task

{What to do}

## Context

{Why, links to relevant specs or Qdrant queries}

## Success Criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Estimated Time

{Hours or days}
```

This format exists because a task file that is missing any of these sections creates predictable problems: a task without context produces work that is technically correct but strategically wrong; a task without success criteria has no definition of done; a task without priority sequencing leads to agents working on low-value items while high-value items wait.

The template is both a prompt output format and a constraint on the Team Lead's thinking. Writing the "Context" section forces the Team Lead to explain *why* the task exists, which frequently surfaces assumptions that need to be made explicit.

### Pattern 3: Qdrant as Context Pre-assembly

For complex tasks, the Team Lead's job is to assemble the context the receiving agent will need and either include it in the task file or load it into the `planning-docs` Qdrant collection and include the collection reference.

The decision to use Qdrant vs inline context depends on size: small decisions and key constraints go inline; large specification documents go to Qdrant with a query hint. This is the "pre-assembly" model — the Team Lead assembles context so the receiving agent does not have to.

The alternative (self-assembly) — where the receiving agent queries Qdrant itself — is also supported, but requires the Qdrant collection to be current and the agent to have a working query pattern. Pre-assembly is more reliable when context is large or complex; self-assembly is more efficient when the receiving agent already knows what to look for.

### Pattern 4: Backlog as Living State

The backlog (`planning/team-tasklist.md`) is not a static list — it is the Team Lead's primary output. Every session involves:

1. Reading the current backlog state
2. Checking the incoming queue for completed work notices, escalations, and new requests
3. Updating task statuses based on received notices
4. Moving completed tasks to the archive
5. Identifying and surfacing blockers
6. Delegating new work or unblocking existing tasks

The key discipline: **the backlog must reflect actual state, not optimistic state.** A task that is blocked should be marked blocked with the blocking condition explicit. A task that is complete should be archived. A Team Lead whose backlog shows everything as "In Progress" is not tracking reality — it is managing appearances.

---

## Prompt Patterns That Work Well for This Role

### 1. Activation Handover Document

When a new Team Lead instance activates (whether due to context reset or role handover), it reads a handover document that captures:

- Current state of the backlog (~20 active tasks in the Podzone case)
- Recent strategic decisions and their rationale
- Outstanding blockers and their dependencies
- The delegation model (which agents handle which work types)
- First session checklist

This is a prompt engineering pattern for continuity: the handover document is essentially an external memory system. It converts the stateless nature of each LLM session into something that feels stateful — the agent's "working memory" persists between sessions as structured markdown.

### 2. Decision Authority Mapping

The Team Lead operates with an explicit decision authority map:

**Can decide unilaterally:**
- Route tasks to agents
- Archive completed tasks
- Create detail files
- Break down high-level work into tasks
- Edit `planning/team-tasklist.md`

**Cannot decide — must escalate:**
- Creating new agent roles (HR role)
- Infrastructure decisions (Cluster Operator)
- Security / architecture decisions (escalate to Trismagistus/Martin)
- Resource allocation requiring human approval

This authority map prevents a common failure mode for orchestrating agents: the agent that, when it cannot route a problem, attempts to solve it itself. Explicit authority limits create clear escalation triggers.

### 3. Budget Awareness as a Routing Constraint

The Team Lead runs on API tokens with a real cost. This shapes its routing decisions: tasks that can be done by the Developer/Coder (subscription, zero marginal cost) should always go to the Developer/Coder, not be processed by the Team Lead directly.

In practice:
- Documentation work → Developer/Coder
- Code review and implementation → Developer/Coder
- Infrastructure audit → Cluster Operator
- Architecture decisions → Claude Web (Architect)
- Policy / resource decisions → escalate to Martin

The Team Lead's own work is primarily *routing* and *decomposition* — the high-leverage activities that unblock other agents. Time spent by the Team Lead writing documentation or reviewing code is wasted API cost.

---

## Failure Modes and How to Avoid Them

### Failure Mode 1: Vague Task Delegation

A task file that says "implement the cluster registry feature" is not a task — it is a wish. The receiving agent will either request clarification (adding a round-trip), make assumptions (producing work that misses the intent), or attempt everything at once (producing a large, hard-to-review change).

**Mitigation:** The task template enforces a success criteria checklist. If the Team Lead cannot write specific, checkable success criteria, the task is not sufficiently decomposed.

### Failure Mode 2: Stale Backlog

A backlog that is not updated when tasks complete or become blocked diverges from reality. The Team Lead then allocates work based on a false model of what is in flight.

**Mitigation:** The session start protocol always processes the incoming queue before making any backlog decisions. Completed work notices, escalation documents, and blocker reports all update the backlog before new delegation happens.

### Failure Mode 3: Authority Overreach

An orchestrating agent that starts making infrastructure decisions (because the Cluster Operator is slow), or writing code (because the Developer/Coder hasn't responded), is abandoning its role. This produces duplicated work, contradictory outputs, and scope confusion.

**Mitigation:** The authority map is explicit and the team structure states clearly: "You CANNOT make infrastructure decisions — that's Cluster Operator." When the Team Lead is tempted to act outside its scope, the correct output is a follow-up task request, not direct action.

### Failure Mode 4: Context Loss at Session Boundaries

A Team Lead that does not maintain its audit trail carefully will open each new session with no knowledge of what was decided in the previous session, which tasks are in what state, or which agents have been sent which instructions.

**Mitigation:** The audit trail is written at the end of every session, covering: duration, actions taken, decisions made, escalations raised, and next session focus. This is the session's terminal output — as important as any task delegation.

---

## Key Principles for Team Lead Prompt Design

| Principle | Why it matters |
|-----------|---------------|
| Decomposition over delegation of vague direction | Vague tasks produce unpredictable outputs |
| Structured task templates are constraints on thinking | Writing the template forces clarity before delegation |
| Authority limits create clear escalation triggers | Agents that expand their own authority produce uncontrolled outputs |
| Budget routing is an explicit constraint | High-cost agent sessions should not do work a zero-cost agent can do |
| The audit trail is a memory prosthetic | Each session is stateless; the audit trail makes the role effectively stateful |
| Backlog must reflect actual state | An optimistic backlog is a decision-making liability |

---

## Check Your Understanding

1. You receive a high-level directive: "The platform needs better observability." Decompose this into at least three tasks using the 4-level hierarchy, including owner, deliverable, and dependencies for each.

2. Write a task delegation document for the Developer/Coder agent to implement a new API endpoint. The endpoint should return the health status of a named Kubernetes cluster. Include all required sections of the task template.

3. An agent sends an escalation to the Team Lead's incoming queue: it has discovered that the Qdrant collection is stale and missing context for a task it has been assigned. What should the Team Lead do, and what is the output?

4. The Team Lead is considering reviewing a code change directly because the Developer/Coder is slow to respond. What should it do instead, and why does the alternative matter even if the Team Lead is technically capable of the review?

5. After a two-week break in active sessions, the Team Lead opens a new session. Describe the first five steps of its session start protocol and what output it should produce before delegating any new work.
