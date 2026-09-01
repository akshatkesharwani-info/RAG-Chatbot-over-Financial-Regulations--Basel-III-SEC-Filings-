# RAG Chatbot over Financial Regulations (Basel III / SEC Filings)

**Question asked:** *What is the minimum Common Equity Tier 1 capital ratio under Basel III?*

**Answer returned, verbatim from the run:**

> The minimum Common Equity Tier 1 (CET1) capital ratio is **4.5% of risk-weighted assets**. This is
> stated in Excerpt 1 (Section 1: Minimum Capital Requirements).

That's a compliance chatbot doing its job correctly — retrieving the exact clause, citing where it
came from, and not making anything up.

## Why RAG and not just "ask an LLM"

An LLM asked cold about Basel III will confidently hallucinate a plausible-sounding number. This
system instead:

1. Chunks the regulation text into overlapping ~300-word pieces
2. Embeds every chunk locally with `sentence-transformers` (`all-MiniLM-L6-v2` — free, runs on CPU)
3. Indexes them in FAISS for fast nearest-neighbor search
4. Retrieves the top-k relevant chunks for a question
5. Feeds *only those chunks* to a free LLM and forces it to answer from them, citing the excerpt

If the answer isn't in the retrieved text, the model is instructed to say so rather than guess.

## Confirmed working end to end

In the actual run, Cerebras returned a `402 payment required` (free quota used up) and the pipeline
automatically continued on Groq — no crash, no manual intervention, correct answer still returned.
This is the exact behavior the fallback was designed for.

## Stack — all free

- `sentence-transformers` (local embeddings)
- FAISS (`faiss-cpu`, local vector search)
- `call_llm()` — Cerebras → Groq → local Ollama, auto-fallback
- `pdfplumber` for reading regulation PDFs

## Run it

Upload a regulation PDF (or don't — it falls back to a small sample regulation text so the notebook
still runs). Provide free Cerebras/Groq keys when prompted, or skip to use local Ollama.

---
*Educational project. Not legal or compliance advice — verify against the primary source.*
