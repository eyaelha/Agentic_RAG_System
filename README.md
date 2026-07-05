Agentic RAG System — Apple 10-K Filings
An agentic Retrieval-Augmented Generation (RAG) system, built with LangGraph, that answers questions about Apple Inc.'s annual reports (10-K filings) for fiscal years 2021–2025, sourced directly from SEC EDGAR.

How it works
Document base. The five most recent Apple 10-K filings are downloaded automatically from SEC EDGAR's public API (data.sec.gov, no key required). Hidden iXBRL metadata is stripped from the HTML before text extraction. Each filing is split by its "Item" section headers (Business, Risk Factors, MD&A, etc.), and any section still too long is further split into fixed-size chunks (1500 characters, 200 overlap). This produces 1073 chunks, each tagged with its filing year and section name.

Embeddings and storage. Chunks are embedded locally with sentence-transformers (all-mpnet-base-v2), not the Gemini embeddings API, after that API's free-tier daily quota was exhausted during development, and stored in a local Chroma vector store.

Tools. Two tools are exposed to the agent:

retrieve_10k: semantic search over the vector store, with optional filters by filing year and/or section.
calculator: evaluates numeric expressions with numexpr (not Python's eval, for safety).
Graph (LangGraph). A ReAct-style loop: an agent node (Gemini with tools bound) decides whether to answer or call a tool; a tools node executes tool calls; a conditional edge loops the agent back until it produces a final answer. State is the running message list; conversation memory is handled by an InMemorySaver checkpointer keyed by session (thread_id).

LLM. Gemini 2.5 Flash via langchain-google-genai. Free-tier quota (20 requests/day) proved insufficient for evaluation; billing was enabled on the Google Cloud project.

Evaluation. 20 questions (10 simple, 10 complex) run through the agent. For each: response time, retrieved chunks, and final answer are logged. A separate LLM-as-judge pass (Gemini, temperature 0) scores retrieval relevance and answer quality (Pass/Fail), with results manually reviewed afterward. Summary: 65% retrieval pass rate, 60% answer pass rate, 9.2s average response time overall.

Requirements
Python 3.12 (developed on Google Colab). Install with:

pip install langgraph langchain-google-genai langchain-community langchain-chroma \
            langchain-text-splitters langchain-huggingface[full] \
            requests beautifulsoup4 html2text numexpr
You also need:

A Google AI (Gemini) API key, set as the GOOGLE_API_KEY environment variable.
Billing enabled on the associated Google Cloud project — the free tier's daily request quota is too low to run the full evaluation.
Repository contents
File	Description
Agentic_RAG_System___Apple_10_K_Filings_.ipynb	Full notebook: data pipeline, agent, graph, evaluation
evaluation_results.json	Raw evaluation results (20 questions, tool calls, judge verdicts)
Known limitations
Content appearing before the first "Item" header in each filing (e.g., cover page details) is not indexed — a chunking gap, not a retrieval-ranking issue.
The agent occasionally invents plausible-but-unverified figures when the exact requested metric isn't a labeled line in the source document.
The evaluation judge shares a model family with the agent it evaluates; manual review found no systematic bias, but this remains a methodological caveat.
