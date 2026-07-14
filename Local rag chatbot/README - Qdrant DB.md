# Local RAG Document Chatbot (n8n + Qdrant + Ollama)

A fully self-hosted Retrieval-Augmented Generation (RAG) chatbot built in n8n — no external AI APIs, no cloud vector DB. Every component (LLM, embeddings, vector store) runs locally.

## Overview

This workflow lets you upload documents through a chat interface, automatically chunk and embed them, store them in a local Qdrant vector database, and then query them conversationally through an AI agent — entirely offline.

## Architecture

**Ingestion pipeline**
1. **Chat Trigger** — accepts file uploads directly from the chat UI
2. **Recursive Character Text Splitter** — chunks documents (500 chars, 50-char overlap)
3. **Default Data Loader** — prepares chunks for embedding
4. **Ollama Embeddings** (`nomic-embed-text-v2-moe`) — generates vector embeddings locally
5. **Qdrant Vector Store** — stores embedded chunks in the `n8n_docs` collection

**Retrieval + generation pipeline**
1. User sends a question via chat
2. **AI Agent** (powered by **Ollama Chat Model** running `llama3.2`) receives the query
3. The agent has a **Qdrant retrieval tool** available, letting it semantically search the same document collection on demand
4. The agent grounds its response in retrieved context and replies

## Tech Stack

- **n8n** — workflow orchestration
- **Ollama** — local LLM (`llama3.2`) + local embeddings (`nomic-embed-text-v2-moe`)
- **Qdrant** — vector database

## Why local?

- Zero API costs (no OpenAI/Gemini/Pinecone billing)
- Full data privacy — documents never leave your infrastructure
- Works offline
- Easy to swap models (Ollama supports many open-source LLMs/embedding models)

## Use Cases

- Internal knowledge base chatbots
- Document Q&A for sensitive/private data
- Offline research assistants
- Prototyping RAG pipelines before committing to paid infra

---

Part of my [AI Automation Projects](https://github.com/cenjoku835/AI-Automation-Projects) portfolio.
