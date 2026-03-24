# Module 5 — Security and Trust

## Overview

Security is not a layer you add to an AI system after it is built — it is an architectural concern you must design in from the start. This module covers the specific threat landscape for AI systems, which differs meaningfully from traditional software security. The prompt injection problem, the trust hierarchy, and the risks introduced by RAG pipelines are all unique to AI systems and require AI-specific mitigations.

---

## Sections

| Section | Title | Estimated time |
|---------|-------|---------------|
| [01](01-trust-boundaries.md) | Trust Boundaries | 40 minutes |
| [02](02-prompt-injection-and-defence.md) | Prompt Injection and Defence | 50 minutes |
| [03](03-secrets-and-pii.md) | Secrets and PII | 40 minutes |
| [03b](03b-pii-professional-services.md) | PII — Professional Services variant *(professional track)* | 30 minutes |
| [04](04-responsible-deployment.md) | Responsible Deployment | 30 minutes |

---

## Learning Objectives

By the end of this module, you will be able to:

- Describe the Control Plane / Data Plane model and apply it to AI system architecture
- Explain direct and indirect prompt injection, and implement defence-in-depth mitigations
- Implement API key management, system prompt confidentiality, and PII scrubbing patterns
- Apply the responsible deployment checklist as a deployment gate
- Map an AI system to the EU AI Act risk tiers

---

## Prerequisites

- Module 1 (Foundations and Prompt Design)
- Module 3 recommended but not required — the security concepts are accessible without implementation experience

---

## Priority Note

This module is marked HIGH PRIORITY. Every developer and architect who deploys AI features should work through sections 01 and 02 as a minimum. Section 04 (the responsible deployment checklist) should be used as a deployment gate for every AI feature going to production.
