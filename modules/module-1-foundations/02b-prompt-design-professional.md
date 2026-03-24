# 1.02b — Prompt Design: Professional Services Track

## Key Insight

A prompt is a structured work instruction, not a sentence. The same four elements that make an engineering prompt reliable make a professional services prompt reliable: a precise instruction, adequate context, clearly delimited input data, and a specified output form. Omitting any one of these forces the model to guess — and in a medico-legal, legal, or consulting context, a guess in the wrong direction can produce a document that misstates a client's position.

---

## The Four Pillars — ICIO

Every effective prompt can be decomposed into four elements. The acronym **ICIO** gives you a checklist you can apply before drafting any prompt:

| Pillar | Letter | Purpose | Professional example |
|--------|--------|---------|----------------------|
| **Instruction** | I | The specific task to perform | "Draft a paragraph comparing the claimant's pre- and post-injury earnings" |
| **Context** | C | Background, constraints, persona, or assumptions | "You are a medico-legal report writer. Apply Paterson grading. All amounts in ZAR." |
| **Input Data** | I | The variable content to process | The earnings schedule, medical reports, and employment history |
| **Output Indicator** | O | The desired format or structure | "One paragraph, formal register, past tense for pre-injury, present tense for current capacity" |

Including all four consistently produces reliable, auditable outputs. Omitting any one tends to produce outputs that are wrong in shape, wrong in register, or unpredictably variable across sessions.

---

## Before and After: Professional Services Prompts

### Example 1: Medico-Legal Earnings Comparison

**Before (unstructured):**

```
Write something about this client's earnings before and after the accident.

Pre-injury salary: R28 500/month
Current capacity: 40% of pre-injury level
```

Problems with this prompt:
- No instruction precision: "something" does not define scope, length, or purpose
- No context: what jurisdiction? what grading system? what is the document for?
- Input data not delimited from the instruction — the model may treat them as one continuous block
- No output indicator: the response format will vary unpredictably across sessions

**After (ICIO structured):**

```
[INSTRUCTION]
Draft a paragraph for a medico-legal report comparing the claimant's pre-injury
earnings to their current residual earning capacity.

[CONTEXT]
You are an experienced medico-legal report writer operating within the South African
legal system. Apply Paterson grading where relevant. Express all amounts in South
African Rand (ZAR). The paragraph must be suitable for submission to the High Court.
Use a formal, objective register. Do not speculate beyond the data provided.

[INPUT DATA]
<earnings_data>
Pre-injury employment: Production Supervisor, Paterson Grade C3
Pre-injury gross monthly earnings: R28 500
Date of injury: 14 March 2022
Current medical finding: 40% residual work capacity (orthopaedic surgeon, report dated
January 2025)
Current employment status: Not employed; claimant reports inability to sustain
full working day due to chronic pain
</earnings_data>

[OUTPUT INDICATOR]
One paragraph. Formal register. Past tense for pre-injury position; present tense for
current capacity. Include the Paterson grade designation. Do not include headings.
```

The structured version tells the model exactly what to produce, who it is writing as, what data to process, and what form the output must take. The result is a paragraph that can be placed directly into a draft report with minimal editing.

---

## The Persona Pattern in Professional Services

Assigning a specific role in the context pillar shapes the model's vocabulary, analytical frame, and default assumptions. This is the **persona pattern**.

```
You are an experienced medico-legal report writer operating within the South African
legal system.
```

Compare the effect of different personas on the same earnings data:

| Weak persona | Strong persona | Effect |
|--------------|----------------|--------|
| "You are helpful" | "You are a medico-legal report writer" | Shifts from generic to professional register |
| "You know about law" | "You are a South African personal injury attorney" | Shifts jurisdiction and procedural framing |
| "You are an analyst" | "You are a consulting actuary assessing loss of earnings" | Shifts toward actuarial methodology and probability language |
| "You understand HR" | "You are a job analyst applying Paterson grading to determine occupational level" | Activates specific grading vocabulary |

The persona pattern is a strong signal, not a guarantee. A model assigned a medico-legal persona can still produce incorrect legal citations or apply the wrong formula. Always review output for correctness. The persona improves the shape and register of the output; professional verification of substance remains your responsibility.

---

## Structural Delimiters in Document Workflows

When you paste client data into a prompt, the model needs to know where your instructions end and the data begins. Without delimiters, the model may treat instructions and data as a single stream — and a client note that happens to contain instruction-like language ("please assess the following", "compare these figures") may unintentionally redirect the model's behaviour.

**Use XML tags to enclose variable client data:**

```
<client_earnings_schedule>
[paste the earnings table here]
</client_earnings_schedule>
```

```
<medical_report_extract>
[paste the relevant section here]
</medical_report_extract>
```

This is especially important when:
- The client data contains imperative language ("please note", "the claimant requests")
- You are processing multiple input documents in a single prompt
- You use the same prompt template across many clients, varying only the data

---

## Worked Example: Pre- and Post-Injury Earnings Paragraph

### Scenario

You are preparing a medico-legal report for a personal injury matter. Your client was a Production Supervisor graded at Paterson C3, earning R28 500 gross per month before sustaining a spinal injury. An orthopaedic surgeon has assessed residual work capacity at 40%. You need a formal earnings comparison paragraph for the damages section.

### The ICIO Prompt

```
[INSTRUCTION]
Draft a paragraph for the damages section of a medico-legal report comparing the
claimant's pre-injury earnings to their current residual earning capacity, and
expressing the monthly earnings loss in ZAR.

[CONTEXT]
You are an experienced medico-legal report writer practising within the South African
legal system. The report is intended for submission in a High Court personal injury
matter. Apply Paterson grading where applicable. All figures must be expressed in
South African Rand. Maintain a formal, objective, third-person register throughout.
Do not draw legal conclusions — your role is factual description and quantification
only.

[INPUT DATA]
<claimant_employment_data>
Pre-injury position: Production Supervisor
Paterson grade: C3
Pre-injury gross monthly earnings: R28 500
Employer: [Client employer name on file]
Date of injury: 14 March 2022
</claimant_employment_data>

<medical_assessment>
Assessing specialist: Orthopaedic Surgeon (report dated January 2025)
Residual work capacity: 40% of pre-injury capacity
Basis: Chronic lumbar pain with radiation; unable to sustain sedentary work for more
than four hours per day; lifting restricted to 5kg maximum
</medical_assessment>

[OUTPUT INDICATOR]
One paragraph. No headings. Formal register. State the Paterson grade. State pre-injury
gross monthly earnings. State the residual capacity percentage and the resulting
estimated monthly earnings capacity. State the monthly shortfall in ZAR. Past tense
for pre-injury facts; present tense for current capacity.
```

### What Good Output Looks Like

A well-formed response to this prompt will:
- Open with the claimant's pre-injury occupational level and Paterson grade
- State the gross monthly earnings figure precisely
- Transition to the post-injury medical finding, citing the specialist and date
- Express residual capacity as a percentage and translate it to a monthly earnings equivalent (40% of R28 500 = R11 400)
- State the monthly shortfall (R17 100) without editorialising
- Use formal, court-appropriate language throughout

The model is doing the mechanical work — structure, register, arithmetic — that consumes time in document production. Your role is to review the output for factual accuracy against the source documents, verify the arithmetic, and ensure the legal framing is appropriate for your jurisdiction and matter.

---

## Practice Exercise

Take one document-drafting task from your current workload — a report section, a client letter, a comparative analysis, a research summary — and construct a prompt for it using the ICIO structure.

Work through each pillar in sequence:

**1. Instruction (I):** Write one sentence that states precisely what the model should produce. Avoid vague verbs like "write something about" or "help me with." Use specific verbs: draft, summarise, compare, extract, classify.

**2. Context (C):** Write 3–5 sentences establishing:
- The persona the model should adopt (your professional domain, jurisdiction if relevant)
- Any constraints or assumptions it must work within
- The register and tone required

**3. Input Data (I):** Identify the variable content for this task. Write the XML tags you will use to delimit it. Consider: are there multiple input documents? Do they each need their own tag?

**4. Output Indicator (O):** Specify:
- Format (paragraph, table, bullet list, numbered list)
- Length (one paragraph, no more than 200 words, five bullet points)
- Tense, person, and register requirements
- Any explicit exclusions ("do not include headings", "do not speculate beyond the data")

Test your prompt. Compare the output to what you would have written manually. Note where the output matches your expectations and where it diverges — each divergence is a signal that one of the four pillars needs refinement.

---

## The Difference Between Prompting and Editing

A common misconception is that the model should produce a final output and the professional should accept or reject it. The more effective working pattern treats AI output as a high-quality first draft that the professional shapes, not a finished product to be approved.

This has two practical implications:

First, your prompts do not need to be perfect on the first attempt. Run the prompt, assess the output, identify which pillar caused the gap, and refine that pillar specifically. This is faster than rewriting the prompt from scratch each time.

Second, your professional judgment is applied at the review stage, not eliminated by AI assistance. The model can produce a formally correct, well-structured paragraph that nonetheless misstates a material fact because the input data was incomplete. The ICIO structure does not substitute for source document verification — it structures the assistance, not the responsibility.

---

## Check Your Understanding

1. A colleague drafts the following prompt: "Summarise this client's situation." Which pillars of ICIO are missing or underspecified, and what would you add to each missing pillar to make this prompt suitable for drafting a formal client report section?

2. You are using the same earnings comparison prompt template across twelve client files, varying only the earnings data each time. Why is it important to use XML delimiters around the input data in this scenario? What could go wrong without them?

3. You run an ICIO-structured prompt and the output is accurate in content but written in an informal, conversational tone that is not suitable for a High Court submission. Which specific pillar needs adjustment, and write the one sentence you would add to fix it?
