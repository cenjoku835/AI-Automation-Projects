# RAG-Powered Document Chatbot (n8n + Pinecone + Gemini)

An AI agent that answers questions about any PDF document by retrieving relevant context from a vector database — instead of relying on the model's memory or risking hallucination. Built entirely in n8n using Google Gemini and Pinecone.

## How It Works

The workflow is split into two independent pipelines:

### 1. Indexing Pipeline
Triggered manually (or on file upload) to ingest a document into the knowledge base:
- **Download file** — pulls a PDF from Google Drive
- **Recursive Character Text Splitter** — chunks the document (1500 characters, 200-character overlap) to preserve context across chunk boundaries
- **Default Data Loader** — prepares the binary file for embedding
- **Embeddings (Google Gemini)** — converts each chunk into a vector embedding
- **Pinecone Vector Store (insert)** — stores the embeddings in a Pinecone index for semantic search

### 2. Chat Pipeline
Triggered on incoming chat messages:
- **Chat Trigger** — receives the user's question
- **AI Agent (Gemini 2.5 Flash)** — the reasoning layer that decides when to search the document
- **Pinecone Vector Store (retrieve-as-tool)** — exposed to the agent as a callable tool, letting it semantically search the indexed document and pull back the most relevant chunks
- The agent grounds its final answer in the retrieved content, reducing hallucination and keeping responses accurate to the source document

## Architecture

```
[Manual Trigger] → [Download PDF] → [Text Splitter] → [Embeddings] → [Pinecone: Insert]

[Chat Trigger] → [AI Agent (Gemini 2.5 Flash)] ⇄ [Pinecone: Retrieve as Tool]
                          ↓
                    Grounded Answer
```

## Tech Stack
- **n8n** — workflow orchestration
- **Google Gemini 2.5 Flash** — LLM for chat reasoning and agentic tool use
- **Google Gemini Embeddings** — text-to-vector embedding model
- **Pinecone** — vector database for semantic search
- **Google Drive** — document source

## Use Cases
- Internal knowledge base / employee handbook Q&A
- Customer support bots grounded in product documentation
- Legal/policy document assistants
- Any "chat with your PDF" application

## Setup
1. Import the workflow JSON into n8n
2. Connect credentials: Google Drive OAuth2, Google Gemini (PaLM) API, Pinecone API
3. Create a Pinecone index and update the index name in both Pinecone nodes
4. Point the **Download file** node at your source PDF in Google Drive
5. Run the indexing pipeline once to populate the vector store
6. Activate the chat pipeline and start querying your document

## Notes
This is a generalized template — swap in any PDF (handbook, contract, manual, report) and the same architecture applies with no logic changes required.

---
Part of a portfolio of n8n automation projects. See other workflows in this repo for email classification and appointment scheduling agents.
