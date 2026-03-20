# 7.01 — The Infrastructure Operator: GitOps Auditing, kubectl Access, and Escalation Patterns

## Key Insight

An infrastructure operator agent has privileged, potentially destructive access — live cluster API, GitOps repositories, and the ability to modify bastion configuration. The prompt engineering challenge for this role is not making the agent capable; it is making the agent appropriately cautious. The system prompt must define scope so precisely that the agent knows what it is *not* authorised to do as clearly as it knows what it is.

---

## What the Role Looks Like in Practice

The Cluster Operator in the Podzone Agent Team is a `claude-sonnet-4-6` instance running in an IDE subscription context. It has:

- Read/write access to multiple GitOps repositories (`cluster09/`, `gitopsdev-apps/`, `podzoneAgentTeam/`)
- `kubectl` access via `~/.kube/config` to five production clusters routed through a bastion host
- Authority to write audit reports, task requests to the Developer/Coder, and escalations to the Team Lead
- No authority to make infrastructure architecture decisions unilaterally

The agent's AGENTS.md (its persistent system prompt / session initialisation file) opens with an explicit scope boundary:

```
GitOpsAPI manages: external ingress (Cloudflare Tunnel → cloudflared → Gateway → public services)
Cluster Operator manages: internal bastion rules (freyr iptables/Apache for 192.168.1.0/24 ↔ 192.168.4.0/24)
```

This kind of explicit boundary is not cosmetic. When a new task arrives asking the agent to handle something adjacent to its scope, the boundary statement is what prevents uncontrolled lateral expansion.

---

## How the Agent Uses AI in Daily Work

### Pattern 1: Repository Audit Before Any Action

Before writing a single line of a report or taking any action, the Cluster Operator reads the current state from source of truth. The session start protocol is:

1. Read `AGENTS.md` for current scope and any scope updates
2. Read all files in `agents/cluster-operator/incoming/`
3. Check `planning/team-tasklist.md` for assigned tasks

This mirrors the four-pillar prompt design (Module 1.02): the agent starts by loading **context** before it processes **instructions**. The AGENTS.md provides persona and constraints; the incoming queue provides the specific task.

### Pattern 2: Evidence-Based Audit Reports

When the agent produces a cluster workload audit, it queries GitOps manifests *and* live cluster state, then reconciles the two. A representative session produced:

- `outgoing/2026-03-16-cluster-workload-audit.md` — structured findings from GitOps manifest analysis
- A live `kubectl` query to verify actual running state against declared state
- A gap list: five specific mismatches between planned and actual state (missing Gateway on gitopsdev, Ollama with no auth, no cloudflared anywhere)

The prompt pattern that drives this is: *ground every claim in observable evidence, then identify the delta between expected and observed.* This is the same pattern as Chain-of-Thought (Module 1.03) applied to infrastructure: reason through the steps visibly before stating a conclusion.

### Pattern 3: Structured Escalation as a Terminal State

When the agent identified decisions it could not make — Ollama authentication strategy, Harbor decommission path, hostname routing for standalone VMs — it did not attempt to make them. It wrote a structured escalation document with:

- A clear statement of the decision required
- The available options (labelled A, B, C) with their implications
- An explicit statement of what is blocked until the decision is made
- A note that current sprint work can proceed without the decision

This is prompt engineering discipline applied to output design. The escalation document is a well-formed output, not a failure state. The agent treats "I cannot decide this" as a valid, expected terminal state — and the format of the escalation ensures the recipient (Team Lead / Martin) has exactly what they need to make the decision and send a response.

---

## Prompt Patterns That Work Well for This Role

### 1. Tool-Use Framing for kubectl

When the agent needs to run a `kubectl` command, it reasons about what it expects to find before running the command, then compares the actual output:

```
Expected: FluxInstance manifest exists for gitopsete and gitopsprod
Action: kubectl get fluxinstance -A --context gitopsete-admin@gitopsete
Observed: No resources found
Conclusion: Flux bootstrap has not completed on gitopsete
```

This explicit expected/observed/conclusion pattern prevents the agent from treating unexpected output as an error to retry. It treats it as signal to investigate.

### 2. Scope Constraints in the Persona Block

The agent's persona statement in AGENTS.md explicitly lists what is *not* in scope:

> Internal access traffic (192.168.1.0/24 → 192.168.4.0/24) that is not externally exposed is NOT in GitOpsAPI scope. Cluster Operator owns bastion configuration for these paths.

When a task instruction arrives that touches something outside this boundary, the agent routes the work rather than attempting it. This is the same pattern covered in Module 3.04 (scope constraints in the system prompt) — the agent's boundary is set at persona level, not task level.

### 3. Output as Input for the Next Agent

Every significant output the Cluster Operator writes is structured so that the next agent in the pipeline (Developer/Coder or Team Lead) can use it without re-reading the raw source. For example, the cluster workload audit produced a clean matrix of apps × clusters that the Developer/Coder used directly to scope GitOpsAPI feature work — no kubectl access required.

---

## Failure Modes and How to Avoid Them

### Failure Mode 1: Scope Creep Under Ambiguous Instructions

An infrastructure agent with broad repo access will attempt to fix things it finds, even when not asked to. Without an explicit scope constraint, a "cluster audit" task can expand to include configuration changes, manifest edits, and even architecture decisions.

**Mitigation:** The AGENTS.md scope section lists specific task IDs and responsibilities. When the agent encounters something outside that list, the instruction is to write a task request to the appropriate agent, not to act.

### Failure Mode 2: Live State vs Declared State Confusion

A cluster audit that reads only GitOps manifests produces findings about declared state. A cluster audit that reads only `kubectl` output produces findings about live state. These are not the same thing — and their divergence is often the most important finding.

**Mitigation:** The agent explicitly labels all findings with their source: "From GitOps manifests" vs "From kubectl query." When it finds a divergence, it flags it specifically rather than reporting whichever source supports the expected conclusion.

### Failure Mode 3: Destructive Actions Without Confirmation

`kubectl` access includes the ability to delete resources, apply manifests, and modify cluster state. An agent that interprets "fix the Flux bootstrap issue" as permission to run `kubectl apply` without human review is a liability.

**Mitigation:** The agent's workflow section specifies that it *writes to outgoing/* and *creates task requests* for implementation work. It does not apply manifests or modify production state except under explicitly approved task scopes. Direct cluster modifications in the audit trail are always documented with the triggering instruction.

### Failure Mode 4: Context Loss Across Sessions

Each new session starts with no memory of previous sessions. An agent that does not maintain its audit trail carefully will re-read large repositories from scratch, repeat work, and lose track of which gaps have already been reported.

**Mitigation:** The audit trail format (date, session, actions, decisions, outstanding items) provides a structured re-entry point. The "Session Start" protocol always reads the audit trail before reading incoming tasks.

---

## Key Principles for Infrastructure Operator Prompt Design

| Principle | Why it matters |
|-----------|---------------|
| Separate declared state from live state explicitly | These are frequently different; conflating them produces wrong conclusions |
| Escalation is a first-class output, not a fallback | Decisions requiring human authority must be escalated promptly and clearly |
| Scope constraints belong in the persona block | Putting them only in task instructions means they apply only to that task |
| Every audit finding needs a source label | "The cluster is unhealthy" without a source is unusable |
| Outstanding items must be preserved across sessions | The audit trail is the agent's memory; it must be maintained actively |

---

## Check Your Understanding

1. An infrastructure agent running a cluster audit discovers that a production service has no authentication configured. It knows how to add authentication via a HelmRelease values patch. Should it make the change? What should the output of this session be?

2. You are designing the system prompt for an infrastructure operator agent that has `kubectl` access to three clusters. Write the scope constraints section — what it is authorised to do, and what it must not do unilaterally.

3. An agent runs a `kubectl` command and gets unexpected output. Walk through the expected/observed/conclusion pattern: what should the agent write down before concluding anything?

4. The agent receives a task: "Audit all services that are exposed via the Apache reverse proxy." How would you structure the output so that a Developer/Coder agent can use it directly to scope a new API feature, without needing to re-query the cluster?

5. After three sessions, the infrastructure agent's audit trail has grown to 800 lines. A new session needs to understand the current state quickly. What should the audit trail format include to make the "re-entry" problem manageable?
