# Hair & Skin Care Knowledge Assistant (RAG + Agent + Memory)

## Project Title
Building a Basic AI Knowledge Assistant with Memory — Hair & Skin Care Product Domain

## Project Description
A conversational AI assistant that answers questions about hair and skin
care products. The knowledge base is built entirely from web-scraped
product data (no manually uploaded PDFs or spreadsheets), converted into
a searchable vector index, and wrapped in a tool-calling agent that
remembers the conversation history across turns.

Unlike a simple fixed retrieval chain, the system is built as an
**agent**: the LLM decides for itself when it needs to search the
product catalog versus when it can answer directly, and it can use the
search tool multiple times within a single turn if needed.

## Key Features
- **Data source:** Live web scraping (Shopify `/products.json` feed),
  not a static uploaded file.
- **Semantic search:** Dense vector retrieval over the full product
  catalog using sentence-transformer embeddings.
- **Agent architecture:** Tool-calling agent (not a fixed chain) that
  decides when to invoke the retriever.
- **Conversational memory:** Full chat history is preserved and used to
  correctly resolve follow-up questions (e.g. "which one is cheapest?").
- **Local LLM:** Runs on a locally hosted Llama model via Ollama — no
  paid API key required.

## Architecture

```
                 ┌────────────────────────────┐
                 │   Product Website (Shopify) │
                 └──────────────┬───────────────┘
                                │  /products.json
                                ▼
                 ┌────────────────────────────┐
                 │   Web Scraper (requests)    │
                 └──────────────┬───────────────┘
                                ▼
                 ┌────────────────────────────┐
                 │  Cleaning & Transformation  │
                 │  (HTML strip, price range,  │
                 │   structured text per item) │
                 └──────────────┬───────────────┘
                                ▼
                 ┌────────────────────────────┐
                 │  Embeddings (HuggingFace)   │
                 └──────────────┬───────────────┘
                                ▼
                 ┌────────────────────────────┐
                 │      FAISS Vector Store     │
                 └──────────────┬───────────────┘
                                ▼
                 ┌────────────────────────────┐
                 │  Retriever Tool (LangChain) │
                 └──────────────┬───────────────┘
                                ▼
      ┌───────────────────────────────────────────────┐
      │   Tool-Calling Agent (ChatOllama / Llama)      │
      │   + ConversationBufferMemory (chat history)    │
      └──────────────┬──────────────────────────────────┘
                                ▼
                        Answer to user
```

## Tech Stack
| Component        | Choice                                   |
|-------------------|-------------------------------------------|
| LLM               | Llama (local, via Ollama)                 |
| Framework         | LangChain (v1.x: `langchain-classic` for legacy agent/memory APIs) |
| Embeddings        | `sentence-transformers/all-MiniLM-L6-v2` (HuggingFace, local) |
| Vector Store      | FAISS                                     |
| Data Source       | Web scraping (Shopify JSON product feed)  |
| Memory            | `ConversationBufferMemory`                |

## Project Status

| Step | Description | Status |
|------|--------------|--------|
| 1 | Scrape products from source website | ✅ Done |
| 2 | Clean & transform into structured documents | ✅ Done |
| 3 | Build embeddings + FAISS vector index | ⏳ In progress |
| 4 | Wrap retriever as an agent tool | ⏳ In progress |
| 5 | Build tool-calling agent with local Llama | ⏳ In progress |
| 6 | Add conversational memory | ⏳ In progress |
| 7 | Testing & evaluation | ⬜ Not started |

## Setup

```bash
uv add requests langchain-core langchain-community langchain-classic
uv add langchain-huggingface langchain-ollama sentence-transformers faiss-cpu
```

Make sure Ollama is installed and running, and the model is pulled:
```bash
ollama pull llama3.2   # replace with your exact local model name
ollama list             # verify the exact name to use in code
```

## Project Structure
```
rag_agent_full/
├── README.md              # This file
├── full_pipeline_en.py     # Complete pipeline: scraping -> agent -> memory
└── faiss_hair_skin_products/   # Saved vector index (created after running)
```

## How to Run
Run the notebook or script cell by cell, in this order:
1. Imports
2. `scrape_all_products()` — pull the catalog
3. `product_to_document()` — convert each product to a Document
4. `build_vectorstore()` — build and save the FAISS index
5. `build_product_search_tool()` — wrap retriever as a tool
6. `build_agent()` — build the agent with memory
7. Test with a question, then a follow-up question, to confirm memory works

## Testing & Evaluation (planned)
- A set of 10–15 test questions (direct + follow-up) will be run against
  the assistant to check answer accuracy and correct use of conversation
  context, as the final step of the project.