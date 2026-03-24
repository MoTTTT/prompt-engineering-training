# 8.01 — AI in the Office Suite

## Key Insight

AI in the office suite is not a single product — it is a category. Some tools come pre-integrated into the applications you already use. Others are separate services you access through a browser. The tools differ in where your data goes, what your organisation has licensed, and what you are permitted to use. Understanding these differences is the foundation of safe and effective AI use at work.

---

## Microsoft 365 Copilot

Microsoft 365 Copilot is Microsoft's AI assistant, built directly into the Microsoft 365 application suite. It uses large language models (primarily developed in partnership with OpenAI) combined with your organisation's content in Microsoft 365 to generate responses that are grounded in your actual work context.

### Where It Appears

| Application | What Copilot Can Do |
|-------------|---------------------|
| **Word** | Draft documents from bullet notes, rewrite sections, summarise long documents, adjust tone |
| **Excel** | Suggest formulas, explain what a formula does, identify patterns in data, generate plain-English summaries of charts and tables |
| **PowerPoint** | Generate slide outlines from a document or prompt, draft speaker notes, suggest layouts |
| **Outlook** | Draft email replies from brief notes, summarise long email threads, flag action items |
| **Teams** | Summarise meeting recordings, extract action items from meeting transcripts, answer questions about what was discussed |

### What Makes It Different From a Standalone AI Chat Tool

The key distinction is that Copilot has access to your Microsoft 365 content — emails, documents, Teams messages, SharePoint files — subject to your existing permissions. When you ask Copilot in Word to "draft a project update based on last week's emails", it can retrieve relevant emails from your Outlook and use them as context. A standalone AI chat tool cannot do this; you would need to copy and paste the content manually.

This integration is powerful, but it also means that Copilot operates within your organisation's Microsoft 365 environment. The data does not leave your organisation's Microsoft tenancy in the way it would if you pasted it into a public chat tool. This is a significant distinction for data protection purposes, which Section 03 addresses in more detail.

### Licensing and Availability

Microsoft 365 Copilot is a paid add-on licence, separate from a standard Microsoft 365 subscription. Not every organisation has purchased it, and not every user in an organisation that has purchased it will have a licence assigned to them. Check with your IT department or Microsoft 365 administrator to find out whether Copilot is available to you.

---

## Other AI Tools Available for Office Workflows

Beyond Microsoft Copilot, several other AI tools are widely used by office professionals:

### ChatGPT (OpenAI)

ChatGPT is available as a web application and as a mobile app. It is a general-purpose AI assistant capable of drafting, summarising, rewriting, and answering questions across a wide range of topics. ChatGPT has a free tier and a paid tier (ChatGPT Plus); enterprise accounts (ChatGPT Team and Enterprise) include data isolation and privacy controls that the free tier does not.

ChatGPT does not have access to your organisation's files unless you upload them directly into a conversation.

### Claude (Anthropic)

Claude.ai is Anthropic's consumer AI assistant, available through a browser. Like ChatGPT, it is a general-purpose assistant capable of document drafting, summarisation, analysis, and question answering. Claude has a free tier and a paid tier; enterprise accounts include additional controls. Claude does not access your organisation's systems; you interact with it by typing or pasting content into the chat window.

### Google Gemini in Google Workspace

Google Gemini is Google's AI assistant, integrated into Google Workspace applications — Google Docs, Sheets, Slides, and Gmail — in a manner analogous to how Microsoft Copilot integrates with Microsoft 365. If your organisation uses Google Workspace, Gemini may already be available depending on your organisation's subscription tier. Like Copilot, it can access your Workspace content subject to your permissions.

---

## Cloud-Hosted vs On-Premises: Where Your Data Goes

This is the single most important distinction for safe use of AI tools at work.

**Cloud-hosted AI tools** process your content on the provider's servers, somewhere on the internet. When you type or paste content into ChatGPT, Claude.ai, or the consumer tier of any similar tool, that content is transmitted to the provider's infrastructure, processed there, and a response is sent back to you. The content has left your device and your organisation's network.

**On-premises or private deployment options** process data on your organisation's own infrastructure. Microsoft 365 Copilot, configured through your organisation's tenancy, keeps data within your Microsoft 365 environment. Some organisations also deploy AI tools on their own servers so that no data leaves their building or cloud infrastructure at all.

The practical implication:

| Tool type | Where data goes | Appropriate for |
|-----------|----------------|-----------------|
| Consumer chat (free tier) | Provider's public cloud | Non-sensitive, non-confidential work only |
| Enterprise chat (with DPA) | Provider's cloud under contract | Depends on your organisation's policy |
| Microsoft 365 Copilot (M365 tenancy) | Your organisation's Microsoft environment | As permitted by your policy |
| On-premises AI | Your organisation's own infrastructure | Maximum data sovereignty |

The golden rule: if the content would require permission to share with a colleague outside your department, treat it as content that requires permission to share with an AI tool.

---

## How to Identify Approved Tools

Your organisation may have a list of approved AI tools — tools that have been evaluated for data protection compliance, procured under appropriate contracts, and assessed against your regulatory requirements. Using an unapproved tool with work content, even with good intentions, is a compliance event in most organisations with a formal AI use policy.

**Steps to identify what is approved:**

1. Ask your manager or IT department whether your organisation has an AI use policy. If one exists, read it before using any AI tool for work tasks.
2. Check whether Microsoft 365 Copilot is available to you — your IT administrator can confirm.
3. If you want to use a tool that is not on any approved list, raise it with your IT or data protection function before use. Do not assume approval — request it.
4. When in doubt, do not use a cloud AI tool with content that identifies specific clients, patients, employees, or commercially sensitive matters.

---

## A Mental Model: The Capable Colleague

The most useful mental model for working with AI tools is to think of the AI as a capable colleague who needs clear, specific instructions.

A capable colleague can draft a first version of a document if you explain clearly what the document is for, who will read it, what it should cover, and what tone is appropriate. The same colleague will produce an unhelpful draft if you hand them a pile of notes and say "write something."

A capable colleague can summarise a long report — but they need to know what you want from the summary: the key findings, the action items, the risks, or a one-page overview for a specific audience.

A capable colleague makes mistakes. They might get a fact wrong, misremember a number, or misunderstand what you wanted. You review their work before it goes out.

This mental model has three practical implications:

**Be specific.** Vague instructions produce vague results. "Help me with this document" is less useful than "Rewrite this paragraph for a non-technical audience, keeping it under 100 words."

**Provide context.** The more context you give, the more useful the output. If the document is a client-facing report for a financial services firm, say so. If the tone needs to be formal, say so.

**Always review.** AI output is a starting point, not a finished product. Check facts against your source documents, verify any numbers, and ensure the register is appropriate before using the output.

The four-pillar ICIO structure introduced in Module 1 (Instruction, Context, Input Data, Output Indicator) applies directly to office workflows. Section 02 of this module shows how to apply it to Word, Excel, and PowerPoint tasks.

---

## Check Your Understanding

1. You have a standard Microsoft 365 subscription for work. You want to use Microsoft 365 Copilot to draft a meeting summary. What would you need to check before assuming this feature is available to you?

2. A colleague tells you they use the free tier of ChatGPT to help draft client reports, and that it saves a lot of time. What question would you ask before deciding whether to do the same in your own role, and why?

3. You need to summarise a long internal strategy document. You have access to Microsoft 365 Copilot in Word, the free tier of Claude.ai, and the free tier of ChatGPT. What is the key difference between using Copilot in Word and using one of the browser-based tools for this task, from a data handling perspective?
