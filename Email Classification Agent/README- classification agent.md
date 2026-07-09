# AI-Powered Email Classification & Routing Agent (n8n)

An n8n workflow that automatically reads incoming Gmail messages, classifies them by department, notifies the right team, and logs every decision — using a **hybrid classification engine** that only calls an LLM when simple keyword matching isn't confident enough.

## Why this exists

Most inboxes need triage: sales inquiries, support tickets, HR requests, invoices, and ops issues all land in the same place. Manually routing them is slow, and running every single email through an LLM is wasteful. This workflow solves both problems with a two-stage classifier.

## How it works

```
Gmail Trigger (polls every 1 min)
        │
        ▼
Rule-Based Classifier  ──► keyword scoring across 5 departments
        │
        ▼
Confident? ──No──► Gemini 2.5 AI Classifier ──► Parse AI Response
        │Yes                                          │
        ▼                                              ▼
        └──────────────► Merge Classification ◄────────┘
                                  │
                                  ▼
                     Switch: Route by Department
                                  │
        ┌───────┬───────┬────────┼────────┬───────────┬──────────┐
        ▼       ▼       ▼        ▼        ▼           ▼          
     Sales    CS       HR    Finance  Operations   Other/Unclassified
        │       │       │        │        │           │
        └───────┴───────┴────────┴────────┴───────────┘
                                  │
                                  ▼
                     Log to Google Sheets (audit trail)
```

### 1. Trigger
- **Gmail Trigger** polls the connected inbox every minute for new messages.

### 2. Stage 1 — Rule-Based Classification (fast, free)
- A Code node scores the email subject + body against keyword dictionaries for **Sales, Customer Service, HR, Finance, and Operations**.
- The department with the highest keyword hit count wins. If nothing matches, it's flagged `Other`.

### 3. Stage 2 — AI Fallback (only when needed)
- An **IF node** checks whether the rule-based pass returned `Other` (i.e., low confidence).
- Only those ambiguous emails are sent to **Gemini 2.5 Flash** via a LangChain Chain LLM node, with a tightly scoped prompt asking for a single department label.
- A parser node cleans and validates the AI's response against the known department list, falling back to `Other` if the response is unusable.

This two-stage design keeps the workflow **cheap and fast for the majority of emails**, while still catching edge cases an LLM handles better than keyword matching.

### 4. Merge & Route
- A **Merge node** recombines the rule-classified and AI-classified branches into a single stream.
- A **Switch node** routes each email to one of six paths based on final department.

### 5. Notify
- Each department has a dedicated Gmail notification node that sends a formatted alert (with an emoji tag, e.g. `📧 [SALES]`) to that team's inbox, including sender, subject, timestamp, classification method, and a body preview.
- Unclassified emails go to an admin inbox for manual review.

### 6. Log
- Every classified email — regardless of department — is appended to a **Google Sheet** with timestamp, message ID, sender, subject, department, classification method (`rule-based` vs `ai`), and a body preview. This gives visibility into how often the AI fallback is triggered, which is useful for tuning the keyword dictionaries over time.

## Tech Stack

- **n8n** — workflow orchestration
- **Gmail API** — trigger + notifications
- **Google Gemini 2.5 Flash** (via LangChain node) — fallback classification
- **Google Sheets** — logging / audit trail

## Setup

1. Import the workflow JSON into your n8n instance.
2. Connect Gmail OAuth2 credentials to the Trigger and all Notify nodes.
3. Connect a Google Gemini (PaLM) API credential to the Chat Model node.
4. Replace the placeholder team email addresses (`sales-team@yourcompany.com`, etc.) with your real distribution lists.
5. Replace `REPLACE_WITH_YOUR_GOOGLE_SHEET_ID` in the Google Sheets node with your own sheet ID, and create a sheet named `Email Log` with matching column headers.
6. Activate the workflow.

## Possible extensions

- Add a feedback loop so misclassified emails (corrected manually) retrain/expand the keyword dictionaries.
- Add Slack notifications alongside/instead of email.
- Track AI-fallback rate over time as a proxy for rule-based classifier accuracy.

---

Part of my ongoing work building production-grade n8n automation workflows.
