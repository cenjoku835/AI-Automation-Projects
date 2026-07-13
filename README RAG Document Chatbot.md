# 📄 RAG Document Chatbot

A retrieval-augmented generation (RAG) system built in **n8n** that lets you upload a PDF and chat with its contents in natural language — no manual searching through pages required.

---

## 🧠 Overview

This workflow indexes a PDF document into a vector database and exposes it through a conversational AI agent. The agent only queries the document store when a question actually requires it, rather than retrieving context on every single message.

It's split into two independent pipelines so that indexing and querying never interfere with each other:

1. **Ingestion Pipeline** — chunks, embeds, and stores a PDF
2. **Conversational Retrieval Pipeline** — answers user questions using an AI agent with retrieval as a tool

---

## 🏗️ Architecture

### 1. Ingestion Pipeline (Manual Trigger)

| Step | Node | Function |
|------|------|----------|
| 1 | **Manual Trigger** | Starts the indexing process on demand |
| 2 | **Google Drive – Download File** | Downloads the target PDF |
| 3 | **Recursive Character Text Splitter** | Splits the document into overlapping chunks (200-token overlap) to preserve context across boundaries |
| 4 | **Default Data Loader** | Prepares the binary PDF data for embedding |
| 5 | **Embeddings – Google Gemini** | Converts each chunk into a vector embedding |
| 6 | **Supabase Vector Store (Insert)** | Stores the vectors + text into a `documents` table |

### 2. Conversational Retrieval Pipeline (Chat Trigger)

| Step | Node | Function |
|------|------|----------|
| 1 | **Chat Trigger** | Listens for incoming user messages |
| 2 | **AI Agent** | Orchestrates the conversation using Gemini as its LLM |
| 3 | **Google Gemini Chat Model** | Powers the agent's reasoning and responses |
| 4 | **Embeddings – Google Gemini** | Embeds the incoming query for similarity search |
| 5 | **Supabase Vector Store (Retrieve-as-Tool)** | Exposed to the agent as a callable tool — the agent decides when to search the knowledge base |

---

## ⚙️ Key Design Choices

- **Decoupled indexing and querying** — new documents can be re-indexed at any time without touching or disrupting the live chat flow.
- **Retrieve-as-tool pattern** — instead of a rigid "always retrieve" chain, the agent autonomously decides whether a given question needs a knowledge base lookup, reducing unnecessary vector searches and latency.
- **Chunk overlap (200 tokens)** — ensures continuity of meaning across chunk boundaries, improving retrieval quality for answers that span multiple chunks.

---

## 🧰 Stack

- **n8n** — workflow orchestration
- **Google Drive** — document source
- **Google Gemini** — embeddings + chat model
- **Supabase Vector Store** — vector database for storing and retrieving document embeddings
- **LangChain (n8n nodes)** — agent, tool, and retrieval abstractions

---

## 🚀 Setup

1. Import `rag-document-chatbot.json` into your n8n instance.
2. Configure credentials:
   - Google Drive OAuth2
   - Google Gemini (PaLM) API
   - Supabase API
3. In Supabase, create a `documents` table with a matching `match_documents` function for vector similarity search (pgvector extension required).
4. Update the **Download file** node with the Google Drive file ID of the PDF you want to index.
5. Run the **Manual Trigger** once to index your document.
6. Use the **Chat Trigger** webhook (or n8n's built-in chat UI) to start asking questions about the document.

---

## 📌 Notes

- Credentials and IDs in this JSON have been sanitized — replace all `YOUR_*` placeholders with your own values before importing.
- This workflow is designed to be document-agnostic; swap in any PDF via the Google Drive node.

---

Part of my growing library of n8n AI automation workflows — see the rest of the portfolio in this repo.
