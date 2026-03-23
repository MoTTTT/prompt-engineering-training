# Session Report — Norma Session 001
**Date:** 2026-03-22
**Reviewer:** Norma (on behalf of Martin / MoTTTT)
**Tutor:** Alex (AI persona)

---

## Session Outcome

Productive session. Norma had no prior AI usage and a clear, practical goal: workflow optimisation for medico-legal report drafting. She engaged well with Socratic prompting, produced a relevant practical prompt attempt, and correctly answered conceptual check questions unprompted.

---

## Modules Covered

| Module | Section | Completion |
|--------|---------|------------|
| 1 — Foundations | 1.01 How LLMs Work | Complete |
| 1 — Foundations | 1.02 Prompt Design | Complete |
| 2 — Context and RAG | 2.01 Context Windows | Partial (concepts) |
| 2 — Context and RAG | 2.03 RAG Architecture | Concepts only |
| 5 — Security and Trust | 5.03 Secrets and PII | Applied to practice |

**Next session starting point:** Module 2 in full, then Module 5 — or direct practical application per Norma's priorities.

---

## Participant Performance

- **Hallucination risk:** Identified unprompted that AI answers may be based on incorrect assumptions — strong baseline instinct
- **Four pillars:** Understood quickly; correctly identified gaps in her own prompt attempt
- **RAG "not found":** Immediately and correctly identified this as a feature (honesty over hallucination) — notable for a first session with no prior AI exposure
- **PII concern:** Proactively asked for specific guidance rather than accepting the general principle — shows professional rigour

---

## Review Criteria Assessment

### Preparation Process
The CLAUDE.md → AGENTS.md → progress.md chain worked well for session initialisation. The persona (Alex) activated correctly. The Socratic approach was appropriate for Norma's professional level.

**Issue:** No `progress.md` existed at session start (correct first-session behaviour), but the file was not checked in `.gitignore` — it is now tracked by `.gitignore` per a prior commit. This means progress will not persist across repo clones. Recommend reconsidering whether `progress.md` should be gitignored for a training context.

### Ease of Use
The programme material is well-structured for a non-technical participant when facilitated by Alex. The medico-legal reframing of all examples (Paterson grades, court reports, client PII) was effective and Norma responded well to role-specific anchoring.

**Issue:** The course material examples are heavily developer-focused (Java, Spring Boot, Kubernetes). For non-technical reviewers like Norma, Alex had to translate all examples. Consider adding a non-technical track or role-specific example sets.

### Startup Prompt with Categorised Context
Norma's opening prompt was well-structured with numbered categories. The system handled it correctly — Alex extracted role, challenges, and goals and used them throughout. This worked well.

**Issue:** Martin's reviewer instructions were appended to the same prompt as Norma's context. This caused a dual-audience problem: the instructions were for Alex (the system), not for Norma (the trainee). Consider a separate channel for reviewer/coordinator instructions.

### Training Continuity — Planned Session Close
`progress.md` was written at session end. `session-summary-2026-03-22.md` was also created. These provide good continuity material.

**Issue:** The training-review repo was never populated due to the token problem (see below). This is the primary continuity gap.

### Training Continuity — Unplanned Session Close
`/tmp` was cleared between the session and Martin's follow-up query, losing the local tracking files. This is a structural weakness: local `/tmp` is not durable. All session tracking should write to the workspace directory, not `/tmp`.

**Recommendation:** Update the session start protocol to write tracking files directly to the workspace (e.g. `./training-review-local/`) rather than `/tmp`.

---

## Infrastructure Issues

### Token Not Available
`TRAINING_REVIEW_GIT_TOKEN` was not set at session start. The variable must be exported in the shell **before** VSCode/Claude Code is launched to be available in the subprocess environment. Setting it in a separate terminal after launch does not propagate.

**Fix:** Add to shell profile (`.zshrc` / `.bash_profile`):
```bash
export TRAINING_REVIEW_GIT_TOKEN=your_token_here
```
Or use a `.env` file loaded at workspace open via VSCode settings.

### Repo Recovery
All session data was reconstructed from conversation context by Martin's request at session end. This worked because the session was still active. If the session had closed, this data would have been unrecoverable.

---

## Trainee Feedback (Norma)

*To be solicited — session ended before this was collected.*

---

## Recommendations for Next Session

1. Begin with a brief recap of the four pillars (3 minutes) — she has it but hasn't practised it
2. Move to Module 2 in full — RAG architecture directly solves her "finding prior documents" challenge
3. Do a live prompt exercise: draft a real report section together using the four-pillar structure
4. Module 5 — brief coverage of POPIA implications specifically (she raised this as professionally critical)
5. Consider skipping Modules 3, 4, 6 or covering only the governance summary in Module 6
