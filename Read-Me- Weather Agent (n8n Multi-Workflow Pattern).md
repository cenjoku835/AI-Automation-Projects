# Weather Agent (n8n Multi-Workflow Pattern)

A two-workflow n8n pattern that splits an AI agent's **reasoning** from its **presentation**: a conversational orchestrator agent that delegates weather questions to a specialist sub-workflow, which fetches live data and hands it to a second, narrowly-scoped LLM call for humanization.

## Architecture

```
┌─────────────────────────────┐
│   Workflow 1: Orchestrator    │
│  ───────────────────────────  │
│  Chat Trigger                 │
│    → AI Agent (Gemini)        │
│        ├─ Buffer Window       │
│        │  Memory              │
│        ├─ Wikipedia Tool      │
│        └─ n8n Workflow Tool ──┼──┐
└─────────────────────────────┘  │
                                  │ invokes as tool
                                  ▼
┌─────────────────────────────┐
│  Workflow 2: Weather          │
│  Specialist                   │
│  ───────────────────────────  │
│  Manual Trigger                │
│    → Set Location               │
│    → OpenWeatherMap             │
│    → Gemini (humanize JSON)     │
│    → Extract response           │
└─────────────────────────────┘
```

## How it works

1. A user sends a message to **Workflow 1**'s Chat Trigger.
2. The **AI Agent** node (Gemini 2.5 Flash) reasons about intent, using **Buffer Window Memory** for conversation context, and chooses between two tools:
   - **Wikipedia** — general knowledge lookups
   - **n8n Workflow Tool** — weather queries, delegated to Workflow 2
3. **Workflow 2** fetches live conditions from **OpenWeatherMap** for the requested city.
4. The raw JSON is passed to a second, purpose-built **Gemini** call (`gemini-3-flash-preview`) whose only job is turning structured weather data into a friendly, emoji-flecked sentence.
5. That formatted string is returned up the chain and relayed to the user by the orchestrator.

This separation means the main agent's context never gets polluted with raw API payloads — it only ever sees clean, human-readable tool output. New capabilities (stocks, news, calendar, etc.) can be added later as sibling sub-workflows without touching the orchestrator's core logic.

## Files

| File | Description |
|---|---|
| `workflows/orchestrator-agent.json` | The main chat-triggered AI Agent (Workflow 1) |
| `workflows/weather-specialist.json` | The weather-fetching + humanizing sub-workflow (Workflow 2) |

## Setup

### 1. Import both workflows
In n8n: **Workflows → Import from File** and import both JSON files above.

### 2. Configure credentials
You'll need to set up two credentials in your n8n instance and re-link them on the imported nodes (credential IDs don't carry over between instances):
- **Google Gemini (PaLM) API** — used by both the `Google Gemini Chat Model` node (Workflow 1) and the `Message a model` node (Workflow 2)
- **OpenWeatherMap API** — used by the `OpenWeatherMap` node (Workflow 2)

### 3. Link the sub-workflow tool
In Workflow 1, open the **Call n8n Workflow Tool** node and set `workflowId` to the ID of your imported **weather-specialist** workflow (it currently points at the original instance's ID and won't resolve on a fresh import).

### 4. Activate Workflow 2 as callable
Workflow 2 needs to be saved/active in the same n8n instance so Workflow 1 can invoke it by ID.

### 5. Test
Open Workflow 1's chat trigger and ask something like *"what's the weather in Enugu?"*

## Known limitation / open TODO

The `Edit Fields` node in Workflow 2 currently hardcodes `Location` to `"Enugu"` rather than reading it dynamically from the tool call. To let the agent check weather for **any** city the user names:

1. Open the **Call n8n Workflow Tool** node in Workflow 1 and define an input field (e.g. `city`) under `workflowInputs.schema`.
2. In Workflow 2, change `Edit Fields → Location` to reference the incoming parameter (e.g. `{{ $json.city }}`) instead of the static string.

> **Note:** This version also fixes a small issue from the original build — the `Message a model` node's prompt used to have a hardcoded sample weather JSON pasted directly into the text. It's now templated as `{{ JSON.stringify($json) }}` so it dynamically converts whatever `OpenWeatherMap` actually returns, rather than always describing Enugu's weather from a fixed sample.

## Tech stack

- [n8n](https://n8n.io) (workflow engine)
- [LangChain n8n nodes](https://docs.n8n.io/integrations/builtin/cluster-nodes/) (`@n8n/n8n-nodes-langchain`)
- Google Gemini (`gemini-2.5-flash` for the agent, `gemini-3-flash-preview` for humanization)
- OpenWeatherMap API

## License

MIT — use freely, adapt as needed.
