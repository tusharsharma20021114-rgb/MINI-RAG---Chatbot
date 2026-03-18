#Mini RAG — AI Assistant

> A fully grounded Retrieval-Augmented Generation (RAG) pipeline for the INDECIMAL home construction marketplace, with a deployed chatbot interface powered by **OpenRouter (free)**.

---

## Live Demo

Open `indecimal_rag_chatbot.html` in any browser — no server required.  
Get a **free** OpenRouter API key at [openrouter.ai/keys](https://openrouter.ai/keys), paste it in the banner, and start asking questions.

For the Python backend: run `python/rag_pipeline.py` (requires dependencies below).

---

## Architecture Overview

```
User Query
    │
    ▼
[1] Chunk Documents  ──►  6 sections → ~30 overlapping chunks
    │
    ▼
[2] Embed Query  ──►  TF-IDF vector (frontend JS) / all-MiniLM-L6-v2 (Python)
    │
    ▼
[3] Cosine Similarity / FAISS Search  ──►  Top-k most relevant chunks
    │
    ▼
[4] Prompt LLM with ONLY retrieved context  (via OpenRouter)
    │
    ▼
[5] Grounded Answer  +  Displayed retrieved chunks (transparency)
```

---

## Embedding Model

### Frontend (JavaScript)
- **Method**: TF-IDF + Cosine Similarity (in-browser, zero latency, zero API cost)
- **Why**: No server needed; works fully client-side; fast for small corpora; fully interpretable scores.

### Python Backend
- **Model**: sentence-transformers/all-MiniLM-L6-v2 (22M params, ~80MB, local & free)
- **Why chosen**: Free open-source, runs locally, strong semantic similarity, fast CPU inference (<100ms), 384-dim embeddings.

---

## LLM

- **Model**: mistralai/mistral-7b-instruct:free via OpenRouter (completely free tier)
- **Why OpenRouter?** Single API, 200+ models, OpenAI-compatible format, free tier with no credit card
- **Why Mistral 7B?** Strong instruction-following, free, good RAG groundedness, fast
- **Grounding**: System prompt forbids external knowledge; retrieved context injected with section labels

---

## Document Chunking

- **Chunk size**: ~120 words (JS) / ~300 words (Python), with 25/50 word overlap
- **Why overlap?** Prevents facts from being split at boundaries
- **6 sections**: packages | kitchen_bathroom | doors_windows_painting | flooring | quality_payments | company_journey

---

## Vector Indexing & Retrieval

- **Frontend**: TF-IDF cosine similarity (pure JS, <5ms for 30 chunks)
- **Python**: FAISS IndexFlatIP (inner product on L2-normalized = cosine similarity)
- **Top-k**: Configurable 2–5 (default 3)

---

## Grounding Enforcement

System prompt on every call:
1. Answer ONLY using provided context
2. Say so if context is insufficient
3. Quote specific numbers and brand names
4. No speculation or invented figures

---

## Running Locally

### Frontend
```bash
open indecimal_rag_chatbot.html
```
Enter your free OpenRouter key in the banner.

### Python Backend
```bash
pip install -r requirements.txt
export OPENROUTER_API_KEY="sk-or-v1-..."
python rag_pipeline.py
```

---

## Quality Analysis — 8 Test Questions

| # | Question | Chunk Relevance | Hallucination? | Answer Quality |
|---|----------|-----------------|----------------|----------------|
| 1 | What is the price of the Premier package? | High | None | Exact Rs.1,995/sqft cited |
| 2 | Which steel brand is used in Pinnacle? | High | None | TATA up to Rs.80,000/MT quoted |
| 3 | Flooring options for Infinia living room? | High | None | tiles/granite/marble + Rs.140/sqft |
| 4 | How does Indecimal ensure quality? | High | None | 445 checkpoints, escrow, dashboard |
| 5 | Main door wallet for Essential? | High | None | Rs.20,000 panelled door cited |
| 6 | How are contractor payments handled? | High | None | Escrow model explained |
| 7 | Exterior paint brand for Pinnacle? | High | None | Asian Paints Apex Ultima |
| 8 | Sanitary fittings in Infinia? | High | None | Rs.70,000 Jaquar/Essco cited |

**Observations**: Retrieval is accurate for keyword-rich queries. Mistral 7B respects grounding constraints. No hallucinations on in-scope questions. TF-IDF may miss semantic paraphrases (sentence-transformers handles better). Latency ~1.5-3s via OpenRouter free tier.

---

## Bonus: Local LLM via Ollama

```bash
ollama pull mistral
# Then swap endpoint in rag_pipeline.py to http://localhost:11434/api/chat
```

| Factor | OpenRouter (free) | Ollama (local) |
|--------|-------------------|----------------|
| Answer quality | Good | Good (same model) |
| Latency | 1.5–3s | 0.5–15s (GPU/CPU) |
| Privacy | API call | Fully local |
| Setup | Key only | Install + download |

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Embedding (frontend) | TF-IDF cosine similarity (JS) |
| Embedding (backend) | sentence-transformers/all-MiniLM-L6-v2 |
| Vector store | FAISS IndexFlatIP |
| LLM | mistralai/mistral-7b-instruct:free via OpenRouter |
| Frontend | Vanilla HTML/CSS/JS (single file) |
| Backend | Python 3.10+ |

---

## Getting Your Free OpenRouter Key

1. Sign up free at [openrouter.ai](https://openrouter.ai)
2. Go to **Keys** → **Create Key**
3. Copy your `sk-or-v1-...` key
4. Paste into the chatbot banner and click **Save**

Key is stored in browser `sessionStorage` only — never sent anywhere except OpenRouter's API.
