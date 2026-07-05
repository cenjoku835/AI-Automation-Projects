# AI Appointment Scheduling Agent (n8n)

An n8n-powered conversational agent that lets users check availability and book appointments on Google Calendar entirely through natural chat — no manual calendar juggling required.

---

## 🧠 Overview

This workflow turns a simple chat interface into a fully functional scheduling assistant. Users can ask things like *"Do you have anything free next Tuesday afternoon?"* or *"Book me for 30 minutes on Thursday morning"*, and the agent:

1. Interprets the request using an LLM (Gemini 2.5 Flash)
2. Checks real-time availability on Google Calendar via the `freeBusy` API
3. Books a 30-minute appointment if the slot is open
4. Sends calendar invites to both parties automatically
5. Politely declines or offers alternatives if the slot is unavailable

---

## ⚙️ Architecture

```
Chat Trigger → AI Agent (Gemini 2.5 Flash + Memory)
                     ├── Tool: Get Calendar Availability (Google Calendar freeBusy)
                     └── Tool: Book Appointment (Google Calendar Events)
```

| Node | Purpose |
|---|---|
| **When chat message received** | LangChain chat trigger — entry point for user messages |
| **AI Agent** | Core orchestrator; holds system instructions, timezone rules, and scheduling logic |
| **Google Gemini Chat Model** | LLM backend (`gemini-2.5-flash`) powering the agent's reasoning |
| **Simple Memory** | Buffer window memory so the agent retains context across turns in a conversation |
| **Get calendar availability** | HTTP tool calling Google Calendar's `freeBusy` endpoint to check open slots |
| **Setting up the appointment** | HTTP tool calling Google Calendar's `events` endpoint to create and send invites for a confirmed booking |

---

## ✨ Key Design Details

- **Strict timezone handling** — All date/time values are locked to `Africa/Lagos` (WAT) and must be passed as ISO 8601 / RFC 3339 strings with the `+01:00` offset, eliminating ambiguity or timezone drift during scheduling.
- **Grounded availability checks** — The agent never assumes availability; it only books after a confirmed "free" response from the calendar API.
- **Conversational memory** — A buffer window memory node allows natural back-and-forth ("actually, can we do 3pm instead?") without losing context.
- **Fixed appointment length** — All bookings are standardized to 30 minutes.
- **Graceful degradation** — If a tool call fails or returns an empty response, the agent is explicitly instructed to inform the user rather than fabricate a confirmation.
- **Automatic invites** — Bookings use `sendUpdates: all`, so both the host and the requester receive calendar invites immediately.

---

## 🛠️ Tech Stack

- **n8n** (workflow orchestration)
- **LangChain n8n nodes** (`chatTrigger`, `agent`, `lmChatGoogleGemini`, `memoryBufferWindow`)
- **Google Gemini 2.5 Flash** (LLM)
- **Google Calendar API** (`freeBusy` + `events` endpoints)

---

## 🔑 Setup

1. Import the workflow JSON into your n8n instance.
2. Configure credentials:
   - **Google Gemini (PaLM) API** — for the language model
   - **Google Calendar OAuth2** — for availability checks and event creation
3. Update the calendar ID in the `Get calendar availability` node's request body to match your target calendar.
4. Adjust the system prompt's timezone and business name if deploying for a different context.
5. Activate the workflow and connect it to your chat interface of choice (e.g. a website widget, WhatsApp, or Telegram trigger).

---

## 📌 Notes

- Currently configured for **Africa/Lagos (WAT)** — update the `+01:00` offset and `timeZone` fields if deploying elsewhere.
- Designed as a narrow, tool-constrained agent rather than an open-ended assistant — availability and booking are the only actions it can take.

---

## 📂 Related Projects

Part of the [AI-Automation-Projects](https://github.com/cenjoku835/AI-Automation-Projects) repository — a growing collection of n8n automation builds.
