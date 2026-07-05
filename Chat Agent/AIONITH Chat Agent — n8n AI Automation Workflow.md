# 🤖 AIONITH Chat Agent — n8n AI Automation Workflow

A conversational AI agent built in **n8n**, powered by **Google Gemini 2.5 Flash**, with live web search (geo-targeted to Nigeria) and math capabilities. Built as part of my ongoing work in AI automation under **ce.dev**.

---

## 🧠 What This Workflow Does

This is a chat-based AI agent that can:
- Hold a real conversation with short-term memory of what was said
- Search Google in real time for up-to-date answers (results localized to Nigeria)
- Perform accurate calculations instead of guessing numbers
- Decide **on its own**, using the LLM, what to search for and where — no hardcoded queries

It's essentially a lightweight, self-hosted research assistant — think of it as a mini "Perplexity-style" agent, scoped for Nigeria-relevant results.

---

## 🏗️ Architecture

```
Chat Trigger 
    │
    ▼
AI Agent (orchestrator)
    │
    ├── LLM: Google Gemini 2.5 Flash
    ├── Memory: Buffer Window Memory (conversation context)
    ├── Tool: Calculator
    └── Tool: Google Search (SerpApi, geo-locked to Nigeria)
```

### Node Breakdown

| Node | Type | Role |
|---|---|---|
| **When chat message received** | `chatTrigger` | Entry point — opens a chat webhook for users |
| **AI Agent** | `agent` | Core orchestrator — routes requests to tools/LLM |
| **Google Gemini Chat Model** | `lmChatGoogleGemini` | LLM (Gemini 2.5 Flash) powering reasoning & responses |
| **Simple Memory** | `memoryBufferWindow` | Retains recent conversation turns for context |
| **Calculator** | `toolCalculator` | Handles math operations reliably |
| **Google search in SerpApi** | `serpApiTool` | Live web search, dynamically queried by the LLM, restricted to Nigeria (`gl: ng`) |

---

## ⚙️ Tech Stack

- **n8n** (workflow automation / orchestration)
- **LangChain nodes** (`@n8n/n8n-nodes-langchain`)
- **Google Gemini 2.5 Flash** (LLM)
- **SerpApi** (Google Search API)

---

## 🔑 Setup

1. Import `workflow.json` into your n8n instance.
2. Add credentials for:
   - **Google Gemini (PaLM) API** — for the chat model
   - **SerpApi** — for live search
3. Activate the workflow.
4. Open the chat trigger URL (or embed it) to start talking to the agent.

> No hardcoded prompts — the agent dynamically decides when to search, what to search for, and when to calculate, based on the conversation.

---

## 💡 Use Cases

- Local business / market research assistant (Nigeria-focused)
- Quick fact-checking and live Q&A chatbot
- Lightweight customer-facing assistant with search + math grounding
- Foundation for more advanced multi-tool agents (easy to extend with more tools)

---

## 🚀 Roadmap / Possible Extensions

- Add more tools (e.g., weather, currency conversion, database lookups)
- Swap Buffer Memory for persistent memory (e.g., Postgres/Redis) for long-term recall
- Add guardrails/system prompt for tone and scope control
- Deploy behind Slack/WhatsApp/Telegram trigger instead of raw chat trigger

---

## 👤 Author

Built by **Njoku Chinecherem Emmanuel** ([ce.dev](#)) — AI automation builder, IT Instructor at APTech Enugu, and PGD Computer Science student at ESUT.

If you find this useful, feel free to fork, star ⭐, or reach out for collaboration on AI/automation projects.

---

## 📄 License

MIT — free to use, modify, and build on.
