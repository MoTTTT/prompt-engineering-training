# Improvement Recommendations — Martin — Session 1 (2026-03-24)

For PROJ-013 review cycle.

---

## 1. CLAUDE.md — Coordinator bypass for TRAINING_REVIEW_GIT_TOKEN check

**Observation:** The token check blocks session start for coordinators running on machines that authenticate via SSH key or credential store rather than token. Martin had to explain the situation manually before the session could proceed.

**Recommendation:** Add a coordinator bypass condition to the token check — e.g., check for a `TRAINING_COORDINATOR=true` env var, or detect if the git remote is already authenticated by running a test push. Alternatively, document that coordinators should set a dummy value for the token env var if their machine doesn't need it.

---

## 2. Workspace file — generated path bug

**Observation:** Auto-generated `.code-workspace` files use relative paths that only work when the file remains in its original location. Copying breaks them. Martin had to manually fix the path.

**Recommendation:** Workspace builder should always generate the `.code-workspace` at repo root with `"path": "."`. If the file must be stored elsewhere (e.g., a workspaces catalogue), the builder should substitute an absolute path for the machine it runs on.

---

## 3. Skills not portable across workspaces

**Observation:** Session skills (`session-start`, `session-end`) live in `podzoneAgentTeam/.claude/skills/` and are invisible in workspaces that don't include that repo. Workspace init does not install them.

**Recommendation:** Workspace builder should copy (or symlink) required skills into the target workspace's `.claude/skills/` during init, based on the agent role definition. Skills should be versioned in the team repo and installed at workspace init time, not assumed to be present.

---

## 4. Module delivery — architect-track sequencing

**Observation:** Martin covered applied content from Modules 2, 3, 5, and 6 in one session, driven by a live use case (observability agent). The standard sequential module order would have been inefficient for someone at his level.

**Recommendation:** The `prompt-engineer` or `ai-practitioner` tracks should include an "architect" entry point that starts with Module 3.04 (agentic patterns) and works backwards to foundations only when needed. Use-case-first, module-as-validation rather than module-as-introduction.

---

## 5. Prompt caching — not covered in existing module content

**Observation:** Prompt caching (cache_control, token economics) is highly practical and immediately cost-relevant but does not appear in the current module content. Martin asked for it explicitly.

**Recommendation:** Add a section to Module 6.04 (LLMOps) or Module 3.01 (API fundamentals) covering prompt caching mechanics, cache_control marker placement, and the stable-prefix design pattern. Include worked cost comparison.

---

## 6. Workspace / Claude Code config management — not in curriculum

**Observation:** The pattern of managing Claude Code config (CLAUDE.md, settings.json, skills) in a git repo, and using `.code-workspace` files as session entry points, is highly practical for the target audience but not covered in any module.

**Recommendation:** Add to Module 3.05 (Engineering Workstation Tooling) or create a new section covering: workspace-as-code, skill portability, session automation via `claude --print`, and the Human:Agent:Project task repo pattern.

---

## 7. secretctl and skills authoring — not in curriculum

**Observation:** Setting up and using secretctl (the MCP-based secrets manager) and creating/using
Claude Code skills are both immediately practical for the target audience but absent from all
current modules. These are core operational tools for anyone running an agentic workstation.

**Recommendation:** Extend Module 3.05 (or the new workspace section from Rec #6) to cover:
secretctl installation and vault setup, storing and retrieving credentials via MCP, authoring
a skill (SKILL.md format, placement in `.claude/skills/`), and the skills installation pattern
at workspace init time.

---

## 8. Interactive trainee profile collection — skill opportunity

**Observation:** The current trainee profile is assembled by Alex from free-form context provided
by the trainee at session start. This places cognitive load on the trainee to know what to share,
and on Alex to infer what is missing.

**Recommendation:** Build a `collect-profile` skill that: (1) displays the full list of profile
attributes with field descriptions and examples before asking anything; (2) steps through each
attribute interactively; (3) writes the completed profile to `trainees/{name}/trainee-profile.md`.
This would standardise profile quality, reduce session start friction, and make onboarding
self-service for coordinators running sessions without live Alex support.
