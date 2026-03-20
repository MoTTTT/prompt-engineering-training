# 7.02 — The Developer/Coder: Codebase Navigation, Schema-First Testing, and Cost-Aware Context

## Key Insight

A developer agent working on a real production codebase faces a context window problem that has no equivalent in tutorial exercises. The codebase is too large to load entirely. The test suite has 252 passing tests that must stay passing. The schema has gotchas that are not in the task description. The prompt engineering challenge here is not writing good single prompts — it is managing what goes into context, in what order, and how to avoid expensive mistakes before they happen.

---

## What the Role Looks Like in Practice

The Claude Code (Coder) agent in the Podzone Agent Team operates in an IDE subscription context — Claude Code via CLI — against the `gitopsapi` repository: a FastAPI backend with Pydantic models, pytest tests, GitOps service integrations, and a Helm chart for deployment.

The agent's operational constraints are set in `CLAUDE.md` (the project-level AGENTS.md equivalent):

- **Use Qdrant for context retrieval, not direct file reads.** The agent queries a vector store at `http://localhost:6333` instead of loading large spec files into the prompt.
- **Batch operations.** Multiple changes per turn, not one-at-a-time.
- **Do not narrate.** No "let me explain..." preamble — write code.
- **Check incoming/ before asking.** The answer to a question may already be in a message from another agent.

These constraints are not stylistic preferences. Each one targets a specific failure mode at production scale.

---

## How the Agent Uses AI in Daily Work

### Pattern 1: Schema-First Before Any API Call

Before calling any write endpoint in testing, the agent follows a documented protocol:

1. List all fields for the target object (e.g. `ClusterSpec`)
2. Review each field's type, default value, and constraints against the Pydantic model
3. Verify the test data JSON matches the schema — confirm metadata keys are present
4. Only then make the API call

This is enforced by the `CLAUDE.md` "API-First Testing Protocol" section. The protocol exists because a schema gap discovered after a round-trip failure in Flux or CAPI (the Kubernetes cluster API) can take significant debugging time to diagnose. The prompt engineering lesson: *the right time to check a schema is before making the API call, not after reading an error message.*

A concrete example: `ClusterSpec` has an `ip_range` field with a specific CIDR format constraint. Loading the Pydantic model definition before writing the test catches this. Skipping it produces a 422 validation error — worse, it can produce a 200 with a bad value that fails silently in the GitOps pipeline hours later.

### Pattern 2: Qdrant-First Context Retrieval

The agent's AGENTS.md is explicit:

> **USE QDRANT FOR CONTEXT RETRIEVAL. DO NOT read files from local workspace for context.**

The collections in use:
- `gitopsgui-specs` — requirements, architecture, task breakdowns
- `planning-docs` — tasks, decisions, strategy from the agent team

When a new task arrives, the agent queries Qdrant for relevant spec snippets rather than loading the full specification file. This reduces context consumption by roughly 80% for most tasks — the difference between a 50,000-token spec and a 3×512-token retrieval.

The failure mode this prevents: loading a large spec into context, summarising it, and then having insufficient token budget for the actual implementation work.

When context is missing or stale, the agent escalates to the Team Lead rather than attempting to fill the gap by reading large files:

```markdown
# [CC-NNN] Load Context for <topic>

**From:** Claude Code
**To:** Team Lead
**Priority:** Blocking <TASK>

**Summary:** Need <topic> context loaded to Qdrant (missing/stale).
```

This is a deliberate design choice: the cost of an escalation is lower than the cost of loading wrong or stale context and working from it.

### Pattern 3: The Auth Pattern Gotcha

`CLAUDE.md` contains an explicit call-out for a recurring mistake that is easy to make and hard to debug:

```
## Auth pattern — critical gotcha
`require_role(*roles)` uses a callable class (`_RoleChecker`), not a closure:
    caller: CallerInfo = require_role("build_manager")          # correct
    caller: CallerInfo = Depends(require_role("build_manager"))  # WRONG — double-wraps
```

This is prompt engineering applied to code generation: the session context pre-empts the most common mistake. Without this in the context, a model generating FastAPI route handlers will reach for `Depends()` as the natural pattern — because that is what the FastAPI documentation shows. The `CLAUDE.md` override prevents the model from following the generic pattern when the codebase uses a specialised one.

The general principle: **codebase-specific patterns that diverge from framework defaults must be in the session context, not discovered by reading the code.** By the time the model has read enough code to infer the pattern, the token budget for the actual task has been consumed.

### Pattern 4: Maintaining Test Suite Integrity

With 252 passing tests, the agent operates under a constraint: any change that breaks an existing test is a regression, not a feature. The session context includes the test command and its expected output state.

The agent does not treat test failures as acceptable intermediate states. Before committing any change, the test suite must pass. This means:

- New endpoints need tests before the PR is raised
- Schema changes require updating existing test fixtures
- The test data directory (`tests/test_data/`) has a required structure that maps to API endpoints

The prompt engineering discipline here: **state the invariant (252 passing tests) explicitly in the session context, and treat any violation of it as a blocking issue, not a known trade-off.**

---

## Prompt Patterns That Work Well for This Role

### 1. The Incoming Queue Check Before Starting

The agent's Quick Start protocol:

1. Read AGENTS.md
2. Check incoming: `cat podzoneAgentTeam/agents/claude-code/incoming/*.md`
3. Check team tasklist for assigned tasks
4. Query Qdrant for task context
5. Write code, test, commit

Step 2 prevents the agent from starting work on a task that has already been assigned to another agent, or from missing a decision that supersedes the current task instruction. In practice, the incoming queue frequently contains clarifications, scope changes, or decisions from the Team Lead or Cluster Operator that change what the task requires.

### 2. Multi-Repo Routing Awareness

The codebase manages two repos per cluster (`{cluster}-infra` and `{cluster}-apps`), with a specific routing table for where each type of content belongs:

| What | Repo | Path |
|------|------|------|
| Flux Kustomization entries | `{cluster}-infra` | `clusters/{cluster}/{cluster}-apps.yaml` |
| HelmRelease + HelmRepository | `{cluster}-apps` | `gitops/gitops-apps/{name}/{name}.yaml` |
| Per-cluster values override | `{cluster}-apps` | `gitops/gitops-apps/{name}/{name}-values-{cluster}.yaml` |

This routing table lives in `CLAUDE.md`. Without it, the agent would need to discover the pattern by reading multiple repos — consuming context budget before any implementation work begins. With it, the agent can write to the correct file path on the first attempt.

### 3. Cost Optimisation Rules as Prompt Constraints

The agent operates under five explicit cost rules:

1. Query Qdrant first — do not load full spec files
2. Batch operations — make multiple changes per turn
3. Do not narrate — no preamble
4. Check incoming before asking — maybe the gateway already documented it
5. Use Ollama for boilerplate when available — save API budget for complex logic

These are not stylistic preferences. The team operates under a daily API budget. An agent that reads full specification files instead of querying Qdrant, or that makes one change per turn instead of batching, directly increases operational cost without improving output quality.

---

## Failure Modes and How to Avoid Them

### Failure Mode 1: Writing to the Wrong Repo

The two-repo-per-cluster pattern means that a HelmRelease written to the infra repo instead of the apps repo will not be reconciled by Flux. The manifest is syntactically correct, the commit succeeds, and the error only surfaces when the application fails to appear in the cluster — potentially hours later.

**Mitigation:** The routing table in `CLAUDE.md` is checked before writing any manifest. The agent confirms the target repo and path against the table before creating a file.

### Failure Mode 2: Double-Wrapping Depends

As described above: the framework default pattern (`Depends(require_role(...))`) produces a double-wrap that breaks FastAPI's dependency injection silently for the `_RoleChecker` callable pattern.

**Mitigation:** The anti-pattern is explicitly named in the session context. The agent can generate code correctly on the first attempt rather than discovering the error through a test failure.

### Failure Mode 3: Stale Context from Previous Sessions

The agent has no persistent memory across sessions beyond what is written to files. A session that begins without reading the current incoming queue may work from a task description that has been superseded.

**Mitigation:** The Quick Start protocol makes reading the incoming queue mandatory, step 2 of 6, before any implementation work starts.

### Failure Mode 4: Scope Expansion

The Developer/Coder agent is explicitly constrained:

- Does NOT write to `planning/team-tasklist.md` — Team Lead manages this
- Does NOT write to `cluster09/` — Cluster Operator handles GitOps manifests

Without these constraints, an implementation agent working on a GitOps feature will naturally start writing to the GitOps repo as part of the feature — which is exactly what should not happen in a multi-agent team where the Cluster Operator owns that boundary.

**Mitigation:** Scope exclusions are stated explicitly in the AGENTS.md with a clear rationale. When the agent encounters work that falls outside its scope, it writes a task request to the appropriate agent rather than proceeding.

---

## Key Principles for Developer/Coder Prompt Design

| Principle | Why it matters |
|-----------|---------------|
| Schema review before API calls | Schema gaps caught before round-trip failures are far cheaper to fix |
| Vector store over file reads for context | Reduces token consumption by 80%+ for large-spec tasks |
| Codebase-specific patterns must be in session context | Models default to framework docs, not project conventions |
| Invariants (test suite passing) are stated explicitly | Unstated invariants get violated; stated ones become constraints |
| Incoming queue check is mandatory before starting | Task instructions are frequently superseded by agent messages |
| Scope exclusions must be explicit | Natural expansion into adjacent areas is the default; restriction requires explicit constraint |

---

## Check Your Understanding

1. A developer agent is tasked with adding a new API endpoint to the GitOpsAPI. The Pydantic model for the request body is defined in `src/gitopsgui/models/cluster.py`. What should the agent do before writing the route handler, and in what order?

2. You are writing the `CLAUDE.md` for a codebase that uses a custom authentication decorator that differs from the framework standard. How would you document this in the session context to prevent the model from using the framework's standard pattern?

3. The agent discovers that implementing a new feature requires modifying a GitOps manifest in a repository it is not authorised to write to. What should it produce as its output, and in what format?

4. Why is querying Qdrant preferable to reading the specification file directly, even if the file is available on disk? What token-budget scenario makes the Qdrant approach fail?

5. After completing a feature implementation, the agent runs the test suite and finds that 3 of the 252 tests are now failing. The tests test a different endpoint from the one that was just modified. How should the agent proceed?
