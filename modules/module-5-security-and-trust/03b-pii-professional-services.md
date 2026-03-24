# 5.03b — Secrets and PII: Professional Services Document Workflows

## Key Insight

In professional services, the PII risk is not primarily in the code — it is in the document. When a fee-earner pastes a client letter, an earnings schedule, or a medical report into a cloud AI tool, they have transmitted personal data about a third party to an external service provider. That act has legal, regulatory, and professional consequences that no amount of technical PII scrubbing in the application layer can prevent, because the person using the tool is the application layer. The control you need is behavioural and procedural, not just technical.

---

## What PII Means in a Professional Services Context

PII — Personally Identifiable Information — is any information that can identify a living individual, either on its own or in combination with other data. In professional services workflows, PII appears routinely and often invisibly.

### Categories Common to Professional Services

| Category | Examples in practice |
|----------|---------------------|
| **Client identity** | Full name, ID number, passport number, date of birth |
| **Case or matter details** | Case number, court date, nature of claim, outcome information |
| **Medical information** | Diagnoses, treatment records, specialist assessments, medication |
| **Earnings and financial data** | Salary slips, bank statements, loss of earnings schedules, tax returns |
| **Employment history** | Employer names, performance records, termination reasons |
| **Contact details** | Home address, personal email address, personal phone number |
| **Family and dependant information** | Spouse or children named in documents |
| **Witness and third-party data** | Any other individual named in client documents |

The critical observation is that professional services documents almost always contain PII about people who have not consented to their data being processed by an AI tool. A medico-legal report about a claimant contains the claimant's full name, ID number, medical history, and earnings history. The claimant did not sign up to have that information sent to a cloud service.

---

## De-Identification Before Prompting

De-identification is the process of replacing identifying information with neutral placeholders before a document is passed to an AI tool. After the AI has completed its task, you restore the real values in your own environment.

### How De-Identification Works in Practice

**Step 1: Identify the sensitive fields**

Before pasting anything into an AI tool, read the document and identify every field that could identify an individual. Work systematically from the categories in the table above.

**Step 2: Replace with consistent placeholders**

Replace each identifying value with a bracketed label. Use consistent labels within the document — if the claimant is referred to three times, use the same placeholder each time.

```
Original:
"Mr Thabo Nkosi (ID: 780412 5432 083), employed at Sasol Ltd, was injured on 14 March
2022. Mr Nkosi earned R28 500 per month prior to the incident."

De-identified:
"[CLAIMANT_A] (ID: [ID_001]), employed at [EMPLOYER_A], was injured on [DATE_INJURY].
[CLAIMANT_A] earned R28 500 per month prior to the incident."
```

Note that the earnings figure is retained in this case — R28 500 is not, on its own, identifying. Whether to retain or anonymise figures depends on your judgement about re-identification risk in context.

**Step 3: Draft the prompt using the de-identified version**

Pass only the de-identified version to the AI tool. The model processes placeholders exactly as it processes real names.

**Step 4: Restore values in your environment**

When you receive the AI output, manually replace the placeholders with the real values before finalising the document. Keep a substitution note — a simple list of placeholder-to-real-value mappings — for this purpose. Destroy the substitution note after use.

### What Cannot Be Adequately De-Identified

Some documents are so specific that even after name removal they remain re-identifiable. A medical report describing a unique injury sustained in a widely reported industrial accident, combined with an employer name and an exact earnings figure, may identify the individual even without a name. In such cases, the appropriate decision is not to use a cloud AI tool for that document at all.

---

## POPIA and GDPR: Implications for Document Workflows

### POPIA (South Africa)

The Protection of Personal Information Act 4 of 2013 applies to any person or organisation that processes personal information about South African data subjects. "Processing" includes collecting, using, transmitting, or storing personal information.

**What this means for AI tool use:**

When you paste client information into a cloud AI tool, you are transmitting personal information to a third-party operator. Under POPIA, this requires:

- A **lawful basis** for the processing. For professional services, the most common basis is that processing is necessary to carry out the professional service the client has engaged you for (section 11(1)(b)). However, this basis does not automatically extend to transmitting client data to AI subprocessors — it covers the service itself, not every tool used to deliver it.
- A **responsible party** determination. Your firm is the responsible party. The AI provider is an operator. You remain accountable for what the operator does with the data.
- **Contractual controls on operators**. POPIA section 21 requires that operators are bound by contract to process only as instructed and to implement appropriate security measures. Check whether you have an operator agreement with your AI tool provider before using client data. Most enterprise AI agreements include this; most consumer AI accounts do not.
- **Data subject rights**. Clients have rights under POPIA: the right to know their data is being processed, the right to access it, and the right to object. If you use AI tools to process client data, your firm may need to disclose this in your privacy notice or client engagement letter.

**Special categories under POPIA:** Health and financial data are subject to heightened protection. These are precisely the categories most common in medico-legal and personal injury work. The bar for processing them is higher: consent or specific statutory authorisation is typically required unless a specific condition in section 11 or 26–32 applies.

### GDPR (European Union)

GDPR applies to the processing of personal data about individuals in the EU, regardless of where the processing organisation is located. For South African firms with EU clients or matters, GDPR applies concurrently with POPIA.

The GDPR principles most directly relevant to AI document workflows are:

**Data minimisation (Article 5(1)(c)):** Process only the personal data that is necessary for the specified purpose. Before prompting, ask: does the AI tool actually need the client's name to complete this task? If the task is to improve sentence structure or suggest a paragraph opening, the answer is usually no. Remove what is not necessary.

**Purpose limitation (Article 5(1)(b)):** Personal data collected for one purpose should not be repurposed. If your client provided their medical records to support a legal claim, using those records to train an AI tool would be a purpose they did not consent to. Before using any AI provider, check their data use terms: does the provider use inputs to improve or train models? Most major enterprise-tier agreements explicitly opt out of this; consumer-tier agreements often do not.

**Accountability (Article 5(2)):** The data controller — your firm — must be able to demonstrate compliance. If a regulator or client asks how their personal data was handled in connection with AI tool use, you must be able to answer.

**Lawful basis (Article 6):** Processing must have a lawful basis. For client documents, legitimate interests or contract performance is typically the applicable basis. Neither basis automatically extends to routing client data through an AI subprocessor without a Data Processing Agreement (DPA). Check whether your AI tool provider offers a DPA and whether your account tier includes it.

---

## Ready-to-Use Prompt Template for De-Identified Report Drafting

The following template is designed for medico-legal and legal document workflows. Copy, adapt, and retain it in your firm's prompt library.

```
[INSTRUCTION]
Draft [specify: a paragraph / a section / a summary] for [specify: a medico-legal
report / a client advice letter / a damages schedule] addressing [specify the
specific topic: pre- and post-injury earnings comparison / summary of medical
findings / liability assessment].

[CONTEXT]
You are an experienced [specify: medico-legal report writer / personal injury attorney /
consulting actuary] practising in [specify jurisdiction]. The document is intended
for [specify: High Court submission / client advice / expert determination].
Maintain a formal, objective, third-person register. Do not speculate beyond
the data provided. Do not include legal conclusions unless specifically instructed.
All monetary figures are in [ZAR / GBP / EUR — specify].

[INPUT DATA]
<claimant_information>
Reference: [CLAIMANT_A]
Occupation: [role, Paterson grade if applicable]
Date of birth: [age in years only, or omit entirely]
Date of incident: [DATE_INCIDENT]
</claimant_information>

<earnings_data>
Pre-incident gross monthly earnings: [amount]
Basis: [salary slip / employment contract / SARS assessment — omit employer name]
Post-incident residual capacity: [percentage]%
Basis: [specialist type and report date — omit specialist name]
</earnings_data>

<medical_findings>
[Paste de-identified extract from medical report here]
</medical_findings>

[OUTPUT INDICATOR]
[Specify: one paragraph / bullet list of findings / numbered schedule].
[Specify tense: past tense for pre-incident facts, present tense for current status].
[Specify any exclusions: do not include headings / do not reproduce input data verbatim].
```

**Usage note:** Complete every [bracketed placeholder] before use. The template is incomplete as printed — it is a structure, not a ready-to-submit prompt. The output must be reviewed against source documents before being incorporated into any professional document.

---

## Tool Selection Criteria

Not every AI tool is appropriate for every task. Before using any AI tool with client or matter data, work through the following questions:

### Deployment Model

**Where does the processing happen?** Cloud-hosted tools (ChatGPT web, Claude.ai consumer, Copilot free tier) process your data on external servers. Enterprise-tier agreements typically include DPAs and data isolation. Consumer-tier agreements typically do not. On-premises tools (a locally hosted LLM via Ollama, for example) process data on your own hardware — no data leaves your environment.

### Data Retention

**Does the provider retain your inputs?** Some providers retain conversation history indefinitely by default. Check the provider's data retention policy. Enterprise agreements usually allow you to configure zero-retention or short-retention windows. For client data, zero-retention is strongly preferable.

### Training Data Use

**Does the provider use your inputs to train or improve models?** Consumer-tier agreements from multiple providers include clauses permitting this. Enterprise agreements typically exclude it. This distinction matters significantly under POPIA and GDPR — if client data is used for model training, you have created a new processing purpose that almost certainly has no lawful basis under your client engagement agreement.

### Data Processing Agreement

**Is there a DPA in place?** A DPA (or Operator Agreement under POPIA) establishes the contractual basis for your firm as responsible party/controller and the AI provider as operator/processor. Without one, you cannot lawfully use the tool for personal data in most professional contexts.

### Organisational Approval

**Has your firm approved this tool?** Your firm's data protection officer, risk committee, or technology governance function may have a list of approved tools. Using an unapproved tool with client data — even once, even with good intentions — is a compliance event and a potential professional indemnity exposure.

| Question | Acceptable answer for client data | Flags requiring escalation |
|----------|-----------------------------------|---------------------------|
| Where does processing happen? | Your infrastructure, or provider with DPA | Consumer cloud service, unknown |
| Does provider retain inputs? | No, or short retention window in DPA | Yes, indefinitely |
| Are inputs used for training? | No, explicitly excluded in agreement | Yes, or unclear |
| Is a DPA in place? | Yes, and covers health/financial data | No DPA, or DPA doesn't cover special categories |
| Has the firm approved this tool? | Yes, on approved list | Not on list, or list doesn't exist |

If any question in the last column applies, do not use the tool with client personal data. Escalate to your data protection officer or risk function first.

---

## Check Your Understanding

1. A paralegal wants to use a consumer AI chat tool (not an enterprise tier) to help draft a summary of a claimant's medical history for a personal injury file. What are the two most significant legal risks in doing this, and what is the minimum change that would make the approach acceptable from a data protection standpoint?

2. You are de-identifying a client's earnings schedule before prompting. The schedule contains the client's name, employer name, monthly salary, and the name of the HR manager who signed it. Which fields must be replaced with placeholders, and for which might you use your judgement about whether replacement is necessary?

3. Your firm is evaluating three AI tools for document drafting: Tool A is a consumer chat product with no DPA available; Tool B is an enterprise plan with a DPA that excludes health data; Tool C is a locally hosted model running on the firm's own servers. The work involves medico-legal reports containing health and earnings data. Which tool (or tools) can be used, and why?
