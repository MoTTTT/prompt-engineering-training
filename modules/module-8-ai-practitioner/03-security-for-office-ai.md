# 8.03 — Security for Office AI: Safe Use

## Key Insight

The security risk in AI tool use for office professionals is not a technical vulnerability — it is a decision made at the keyboard. Every time you type or paste content into an AI tool, you make a decision about where that content goes. Most of the harm caused by AI tool misuse in professional environments happens not through malice but through habit: people treating a cloud AI chat window the same way they treat a private document on their desktop. It is not the same. This section gives you the habits that prevent the most common mistakes.

---

## What Must Never Go Into an AI Prompt

Certain categories of information must not be entered into any AI tool unless your organisation has specifically assessed and approved that use with appropriate controls. The default position — before checking your organisation's policy and the specific tool's terms — is do not.

### Client Personal Information

Names, identification numbers, dates of birth, addresses, contact details, and any other information that identifies a specific client or their family members. This applies equally to potential clients whose information you hold.

### Case or Matter Details

Information about specific legal matters, insurance claims, medical cases, financial transactions, or any other professional engagement that is confidential by its nature. The confidentiality obligation to your client does not pause when you use an AI tool.

### Medical Information

Diagnoses, treatment records, medication details, specialist assessments, and any health information about any individual. Medical information is a special category under data protection law — the bar for processing it lawfully is significantly higher than for general personal data.

### Earnings and Financial Data

Salary details, bank account information, financial statements, loss of earnings schedules, and similar data that relates to identifiable individuals. Like medical data, detailed financial information can be used to identify and harm individuals if it reaches the wrong hands.

### Trade Secrets and Commercially Sensitive Information

Unannounced business plans, pricing strategies, merger and acquisition activity, unreleased product information, internal financial projections, and any other information that would give a competitive advantage if disclosed.

### Passwords, Credentials, and Security Information

No password, authentication code, PIN, API key, or access credential of any kind should ever be entered into an AI tool. There is no legitimate use case for this.

---

## The Cloud Boundary: Where Your Data Goes

Understanding what happens when you use a cloud AI tool is the foundation of safe practice.

When you type content into a cloud AI chat window — ChatGPT, Claude.ai, Google Gemini in a browser, or any similar tool — that content is transmitted over the internet to the provider's servers. It is processed on those servers by the model. A response is generated and sent back to you. At every point in that process, the content exists outside your device and outside your organisation's network.

This matters because:

**Provider data retention.** Many providers retain conversation history — sometimes indefinitely, sometimes for a defined period. If you enter client information and that information is retained in the provider's systems, you have caused it to be processed by a third party without a lawful basis for that processing and without the data subject's knowledge.

**Data used for model training.** Some providers — particularly consumer-tier services — include terms that permit them to use your inputs to improve or train their models. If client information you entered is used for this purpose, you have created a processing purpose that your client never consented to and that your professional obligations almost certainly prohibit.

**Security incidents at the provider.** If the provider experiences a data breach, content you have submitted may be part of the breach. You cannot control this once data has left your environment.

**The enterprise exception.** Enterprise-tier agreements from major providers typically include data isolation, zero-data-retention options, and explicit exclusions of training use. If your organisation uses an enterprise-licensed AI tool under such an agreement, some of these risks are mitigated contractually. Section 04 explains what to check.

The takeaway: a cloud AI tool is not a private workspace. It is a conversation with an external service. Treat what you share with it accordingly.

---

## De-Identification: How to Use AI Safely on Sensitive Content

If you need AI assistance with a task that involves sensitive content, de-identification allows you to get the assistance without sharing the identifying details.

De-identification means replacing names and other identifying information with neutral placeholders before pasting content into an AI tool. You work with the de-identified version, and you restore the real information in your own environment afterwards.

### A Practical Example

**Original (do not paste this):**
```
Mr Thabo Nkosi was injured on 14 March 2022 while employed at Sasol Ltd.
His pre-injury earnings were R28 500 per month. The orthopaedic surgeon's
assessment of 15 January 2025 concludes that he has 40% residual capacity.
```

**De-identified (safe to work with):**
```
[CLAIMANT_A] was injured on [DATE_A] while employed at [EMPLOYER_A].
Pre-injury earnings: R28 500 per month. Medical assessment dated [DATE_B]
concludes 40% residual capacity.
```

**What this achieves:** The AI can help you draft the relevant paragraph, analyse the figures, or improve the structure. It cannot identify the individual from the de-identified text, because the identifying information has been removed. You restore the real names and dates once you have the output you need.

**Keep a substitution record while working.** A simple handwritten note or a private document — never an AI tool — recording that [CLAIMANT_A] = Mr Thabo Nkosi is all you need. Destroy it once the work is complete.

### The Limits of De-Identification

De-identification does not make all content safe to use. Some combinations of details identify an individual even without a name — a specific injury sustained in a widely reported incident, combined with an exact salary and a named employer, may be enough. Use your judgment: if a reasonably informed person could identify the individual from the de-identified text, it is not adequately de-identified.

---

## Recognising AI Output That Should Not Be Trusted

AI models can produce confident-sounding output that is factually wrong. This is not a bug or a feature — it is a characteristic of how these systems work. The model generates the most probable continuation of the text, not a verified statement of fact. Understanding this protects you from errors that could affect your work and your clients.

### Hallucination in Factual Claims

The term "hallucination" describes the phenomenon of an AI model generating plausible-looking but incorrect or fabricated content. A model might:

- State a legal principle that does not exist, or get the jurisdiction wrong
- Calculate an incorrect figure and present it confidently
- Describe a regulation that was repealed, or cite a case that was not decided as stated
- Attribute a statement to a person who did not make it

Hallucinated content looks indistinguishable from accurate content in the text. There is no formatting difference, no uncertainty marker, no flag. The only way to catch it is to verify against authoritative sources.

### Fabricated Citations

Models frequently generate references, case citations, journal articles, or web URLs that look real but do not exist. A model asked to support a point with references will produce references that are syntactically correct and plausible — but the cited article, case, or source may never have existed. Never include a citation in professional work without verifying that it exists and says what the model claims.

### Confident but Wrong Answers

A model asked a question it does not know the answer to will generally not say "I don't know." It will produce a plausible answer. If you ask about a specific local regulation, a niche technical standard, or a recent event after the model's training cutoff, the answer you receive will be the model's best inference — which may bear little resemblance to the actual answer.

The practical rule: use AI output for structure, drafting, and analysis of content you have provided. Do not use it as a factual source for information you have not already verified.

---

## A Pre-Prompt Checklist

Before pasting any content into an AI tool for work purposes, run through this checklist. It takes thirty seconds and it prevents the most common mistakes.

**1. Does this content identify any individual?**
Names, ID numbers, addresses, dates of birth, contact details of any person — client, employee, patient, witness, or counterparty. If yes: de-identify before proceeding, or do not use a cloud AI tool.

**2. Does this content contain confidential professional information?**
Case details, transaction details, medical information, financial data, trade secrets, or any other information that is confidential by its professional nature. If yes: check your organisation's policy and tool approval status before proceeding.

**3. Does this content contain organisational security information?**
Passwords, PINs, authentication codes, network addresses, or internal system credentials. If yes: stop. Do not enter this into any AI tool.

**4. Is this tool approved by my organisation for this type of content?**
Check your organisation's AI use policy. If you are unsure, ask before using. Uncertainty is not implicit approval.

**5. Am I happy for this content to be processed outside my organisation's network?**
If the answer is anything other than a clear yes, revisit questions 1–4.

If all five checks pass, proceed. If any check raises a concern, resolve it before entering the content.

---

## Check Your Understanding

1. You receive an email from a client containing their salary details, which they want included in a report you are preparing. You want to use an AI tool to help draft the relevant section. What steps from this section would you take before pasting any content, and in what order?

2. You ask an AI tool to help you draft a paragraph that references a specific section of the Companies Act. The model produces a well-structured paragraph that cites "Section 72(4)" of the Act. What should you do before using this paragraph in a formal document, and why?

3. A colleague says: "It's fine — I just remove the client's name before I paste things in." What is correct about this approach, and what might still be a problem depending on the content?
