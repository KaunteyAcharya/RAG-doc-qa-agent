# Local RAG Document Q&A Agent

A fully local, free, and open-source Retrieval-Augmented Generation (RAG) pipeline for asking questions about your own documents — no cloud APIs, no API keys, no data leaving your machine.

## Stack
- **Orchestration:** [n8n](https://n8n.io/) (self-hosted, Docker)
- **LLM (generation):** [Ollama](https://ollama.com/) running `llama3.2:1b` locally
- **Embeddings:** Ollama running `nomic-embed-text` locally
- **Vector store:** [ChromaDB](https://www.trychroma.com/) (self-hosted, Docker)

## Architecture
1. **Ingestion:** A form-upload trigger accepts a PDF/TXT/DOCX/MD document, splits it into chunks (400 chars, 100 overlap), embeds each chunk with `nomic-embed-text`, and stores the vectors in ChromaDB.
2. **Retrieval + Q&A:** A chat trigger takes a user question, retrieves the top-15 most relevant chunks from ChromaDB, and passes them as context to a locally-run LLM (`llama3.2:1b`) with a strict anti-hallucination system prompt — the model is instructed to answer only from the retrieved context and say so explicitly when it can't find an answer.

## Setup
1. Install [Ollama](https://ollama.com/download) and pull the required models:
   ```
   ollama pull llama3.2:1b
   ollama pull nomic-embed-text
   ```
2. Start ChromaDB and n8n:
   ```
   docker compose up -d
   ```
3. Open n8n at http://localhost:5658, import `rag-workflow.json`, and create your own credentials for the two placeholders it references:
   - **ChromaDB Self-Hosted** (`YOUR_CHROMADB_CREDENTIAL_ID`) — base URL http://localhost:8000
   - **Ollama** (`YOUR_OLLAMA_CREDENTIAL_ID`) — base URL http://host.docker.internal:11434 (or your host's Ollama address)

   n8n will prompt you to re-select a credential for each node showing the placeholder — just pick (or create) your own.
4. The workflow imports as inactive with the chat trigger's "Make Chat Publicly Available" option off (`public: false`). Activate the workflow, then toggle that option on if you want a fixed hosted chat URL, and publish.
5. Upload a document via the form trigger's URL, then ask questions via the chat widget.

## Bring your own document
This repo does not ship a sample document. Use the form-upload trigger to add any PDF/TXT/DOCX/MD file of your own — your document stays entirely on your machine (ChromaDB and Ollama both run locally).

## Notes
- Retrieval quality depends heavily on chunk size and top-k; tune `Recursive Character Text Splitter` and `Vector Store Retriever` to your document type.
- `llama3.2:1b` is intentionally small for fast local inference; swapping in a larger model (e.g. `llama3.2:3b` or `qwen2.5:3b`) improves multi-item reasoning (e.g. correctly pairing list items with their attributes) at the cost of speed.

## License
MIT
