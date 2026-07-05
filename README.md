Agentic RAG System — Apple 10-K Filings
An agentic Retrieval-Augmented Generation (RAG) system built with LangGraph that answers questions about Apple Inc.'s annual reports (10-K filings) for fiscal years 2021–2025, sourced directly from SEC EDGAR.
How It Works
Document ingestion. The five most recent Apple 10-K filings are automatically downloaded from SEC EDGAR's public API (data.sec.gov, no API key required). Hidden iXBRL metadata is stripped from the HTML before text extraction. Each filing is split by its "Item" section headers (Business, Risk Factors, MD&A, etc.); sections that remain too long are further split into fixed-size chunks (1,500 characters, 200-character overlap). This pipeline produces 1,073 chunks, each tagged with its filing year and section name.
Embeddings and storage. Chunks are embedded locally using sentence-transformers (all-mpnet-base-v2) — rather than the Gemini embeddings API, whose free-tier daily quota was exhausted during development — and stored in a local Chroma vector store.
Tools. Two tools are exposed to the agent:

retrieve_10k — semantic search over the vector store, with optional filters by filing year and/or section.
calculator — evaluates numeric expressions using numexpr (rather than Python's eval, for safety).

Graph (LangGraph). A ReAct-style loop: an agent node (Gemini with tools bound) decides whether to answer directly or call a tool; a tools node executes tool calls; a conditional edge routes control back to the agent until a final answer is produced. State is maintained as the running message list, with conversation memory handled by an InMemorySaver checkpointer keyed by session (thread_id).
LLM. Gemini 2.5 Flash, accessed via langchain-google-genai. The free-tier quota (20 requests/day) proved insufficient for evaluation, so billing was enabled on the associated Google Cloud project.
Evaluation. 20 questions (10 simple, 10 complex) were run through the agent. For each, response time, retrieved chunks, and the final answer were logged. A separate LLM-as-judge pass (Gemini, temperature 0) scored retrieval relevance and answer quality (Pass/Fail), with results manually reviewed afterward.
Results: 65% retrieval pass rate · 60% answer pass rate · 9.2s average response time.
Requirements
Python 3.12 (developed on Google Colab). Install dependencies with:
bashpip install langgraph langchain-google-genai langchain-community langchain-chroma \
            langchain-text-splitters langchain-huggingface[full] \
            requests beautifulsoup4 html2text numexpr
You will also need:

A Google AI (Gemini) API key, set as the GOOGLE_API_KEY environment variable.
Billing enabled on the associated Google Cloud project — the free tier's daily request quota is too low to run the full evaluation.

Repository Contents
FileDescriptionAgentic_RAG_System___Apple_10_K_Filings_.ipynbFull notebook: data pipeline, agent, graph, evaluationevaluation_results.jsonRaw evaluation results (20 questions, tool calls, judge verdicts)
Known Limitations

Content appearing before the first "Item" header in each filing (e.g., cover page details) is not indexed — this is a chunking gap, not a retrieval-ranking issue.
The agent occasionally produces plausible-but-unverified figures when the exact requested metric isn't a labeled line item in the source document.
The evaluation judge shares a model family with the agent it evaluates. Manual review found no systematic bias, but this remains a methodological caveat.


Changements principaux :

Formatage cohérent en gras pour les sous-sections
Tableau markdown propre pour les fichiers du repo
Grammaire et fluidité améliorées ("proved insufficient" au lieu de répétitions)
Résultats mis en évidence en une ligne facile à lire
Suppression des répétitions et clarification de certaines phrases (ex: "invents" → "produces")
