# Agent Improvement Recommendations — Session 001 (Norma, 2026-03-22)

*This report is intended for use in the improvement feedback loop by training co-ordinators and programme developers.*

---

## 1. Material Improvements

### 1.1 Non-technical participant track needed
All module examples are developer-focused (Java, Spring Boot, Kubernetes, CI/CD). For non-technical participants (psychologists, lawyers, managers, consultants), Alex must translate every example in real time. This works but adds cognitive load and risks losing the participant.

**Recommendation:** Add a parallel example set for non-technical roles in Modules 1 and 2. A legal/professional services track would serve medico-legal, compliance, and consulting participants directly.

### 1.2 Module 5 PII section needs a professional services variant
Section 5.03 covers PII in software systems. For professionals who handle client data in documents (not code), the controls described (Java PII scrubbers, Kubernetes secrets) are not actionable. A practical protocol for document-based professional workflows is missing.

**Recommendation:** Add a "professional services" sub-section to 5.03 covering: tool selection criteria, de-identification before prompting, POPIA/GDPR implications for document workflows, and a copy-paste prompt template.

### 1.3 Paterson grading and SA labour market context
South African participants working in employment/earnings assessment will always need Paterson grading context. This is not in the current material.

**Recommendation:** Add a South African professional services example set, or at minimum note in Module 1.02 that the output indicator pillar should include jurisdiction-specific schemas (Paterson grades, ZAR, etc.).

---

## 2. Delivery Mechanism Improvements

### 2.1 Token/credential setup must be pre-session
The `TRAINING_REVIEW_GIT_TOKEN` approach requires the variable to be set before the Claude Code process starts. This is an infrastructure dependency that blocked the entire review tracking capability for this session.

**Recommendation:** Document this clearly in the co-ordinator setup guide. Consider an alternative: write session tracking to the workspace and use a post-session push hook, removing the real-time dependency on the token being available.

### 2.2 Tracking files must not use /tmp
Writing session tracking to `/tmp` is not durable. `/tmp` is cleared on reboot and sometimes earlier on macOS. All session artefacts must be written to the workspace directory.

**Recommendation:** Update the session tracking protocol to write to `./training-review-local/` in the workspace. This survives session closure.

### 2.3 Reviewer and trainee instructions in the same prompt
Martin's coordinator instructions and Norma's participant context arrived in the same opening prompt. This is appropriate for this review setup but creates a dual-audience problem — the instructions for the system are mixed with the trainee's onboarding context.

**Recommendation:** Consider a mechanism for coordinator instructions to be delivered via CLAUDE.md or a separate pre-session file, keeping the trainee's opening prompt clean and participant-focused.

### 2.4 End-of-session feedback collection not triggered
Alex did not explicitly solicit trainee feedback before session close. The session wrapped naturally when Norma said "yes we can wrap up" and then immediately moved to a practical task request.

**Recommendation:** Add an explicit end-of-session prompt to AGENTS.md: before writing progress.md, Alex should ask "Before we close — what one thing would you change about this session?" This feeds the improvement loop and ensures the trainee feels heard.

---

## 3. Product Improvements

### 3.1 Session recovery from context is viable but fragile
When the repo was unavailable, all session data was successfully reconstructed from conversation context. This demonstrates that the session context is a reliable recovery source — but only while the session is active.

**Recommendation:** Consider a mid-session checkpoint that writes a structured summary to the workspace every N prompts, not just at session end. This would make unplanned closure recovery automatic rather than manual.

### 3.2 Progress.md is gitignored — inter-session continuity broken across clones
The `.gitignore` file excludes `progress.md`. This means a fresh clone of the repo starts with no progress context. For a training programme where continuity is a key review criterion, this is a significant gap.

**Recommendation:** Either remove `progress.md` from `.gitignore` and commit it, or store progress in the training-review repo (student branch) and load it from there at session start.

### 3.3 Out-of-scope requests handled well but could be scoped
Norma requested help with Mac/iCloud migration and file conversion — legitimate workflow needs but outside the programme. Alex assisted, which was the right call for participant experience. However, this extended the session beyond its training purpose.

**Recommendation:** AGENTS.md could include guidance on out-of-scope practical requests: assist briefly, create an artefact if useful, then explicitly return to the programme. Alex did this naturally but it could be formalised.
