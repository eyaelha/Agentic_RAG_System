Agentic RAG System — Apple 10-K Filings

Un système agentique de Retrieval-Augmented Generation (RAG) construit avec LangGraph, capable de répondre à des questions sur les rapports annuels d'Apple Inc. (10-K) pour les exercices fiscaux 2021–2025, sourcés directement depuis SEC EDGAR.


🧭 Comment ça fonctionne

1. Ingestion des documents


Les cinq derniers dépôts 10-K d'Apple sont téléchargés automatiquement depuis l'API publique de SEC EDGAR (data.sec.gov, aucune clé API requise).
Les métadonnées iXBRL cachées sont supprimées du HTML avant l'extraction du texte.
Chaque dépôt est découpé selon ses sections "Item" (Business, Risk Factors, MD&A, etc.).
Les sections encore trop longues sont ensuite subdivisées en chunks de taille fixe (1 500 caractères, avec chevauchement de 200 caractères).
Ce pipeline produit 1 073 chunks, chacun étiqueté avec son année de dépôt et le nom de sa section.


2. Embeddings et stockage


Les chunks sont vectorisés localement avec sentence-transformers (all-mpnet-base-v2) — plutôt que via l'API d'embeddings Gemini, dont le quota gratuit quotidien a été épuisé pendant le développement.
Les vecteurs sont stockés dans une base Chroma locale.


3. Outils (Tools)

Deux outils sont exposés à l'agent :

OutilDescriptionretrieve_10kRecherche sémantique dans la base vectorielle, avec filtres optionnels par année de dépôt et/ou section.calculatorÉvalue des expressions numériques via numexpr (plutôt que eval de Python, pour des raisons de sécurité).

4. Graphe (LangGraph)

Une boucle de type ReAct :


Un nœud agent (Gemini avec outils liés) décide de répondre directement ou d'appeler un outil.
Un nœud tools exécute les appels d'outils.
Une arête conditionnelle renvoie le contrôle à l'agent jusqu'à production d'une réponse finale.
L'état est maintenu sous forme de liste de messages, avec une mémoire de conversation gérée par un checkpointer InMemorySaver indexé par session (thread_id).


5. LLM


Gemini 2.5 Flash, utilisé via langchain-google-genai.
Le quota gratuit (20 requêtes/jour) s'étant révélé insuffisant pour l'évaluation, la facturation a été activée sur le projet Google Cloud associé.



📦 Stack technique


LangGraph — orchestration de l'agent
LangChain (google-genai) — intégration LLM
Gemini 2.5 Flash — modèle de langage
Sentence-Transformers (all-mpnet-base-v2) — embeddings
Chroma — base de données vectorielle
NumExpr — calculs numériques sécurisés
SEC EDGAR API — source des données
