# Glossary

Key terms in one place. Look here first when something feels fuzzy.

---

**Agent / Agentic RAG** — A RAG variant where an LLM dynamically decides whether to retrieve, which retrieval method to use, and whether to retry. Flexible but slow, expensive, and hard to evaluate.

**BM25** — A classic keyword-based retrieval algorithm. Scores documents based on term frequency (how often query words appear), inverse document frequency (how rare those words are across the corpus), and document length. Strong on exact matches; weak on paraphrases.

**Chunking** — Splitting a long document into smaller pieces (e.g., 500 tokens each) before embedding. Each chunk gets its own vector. Necessary because embedding models have input-length limits and because smaller chunks give more precise retrieval.

**CLAP** — An open audio embedding model. *Contrastive Language-Audio Pre-training.* Trained so that audio and matching text descriptions land near each other in the same embedding space — meaning you can search audio with *text* queries.

**Closed-loop test** — An end-to-end RAG eval where the generated output is fed *back* into the retrieval system and measured against the original query. For Crate: generate a Suno prompt, run it on Suno, embed the result, measure its distance from the seed cluster.

**Cold start** — The problem of recommending a brand-new item (or making recommendations for a brand-new user) when there's no interaction data yet. Content-based retrieval sidesteps this because it only needs the item's content (e.g., audio), not interaction history.

**Collaborative filtering** — A recommendation method based on user-item interactions: "people who liked X also liked Y." Needs thousands of interactions per item to work well. Breaks at AI-music scale where most items have zero interactions.

**Cosine similarity** — A common measure of how close two vectors are. Ranges from -1 (opposite directions) to 1 (identical directions). 0 means unrelated. Used as the "closeness" metric in most vector databases.

**Embedding** — A list of numbers (a *vector*) that represents the meaning, sound, or content of an item. Produced by running the item through an embedding model. Items with similar meaning get similar vectors.

**Embedding model** — A neural network that takes input (text, audio, image) and outputs an embedding. Examples: OpenAI's `text-embedding-3` for text, CLAP and MERT for audio.

**F1 score** — Harmonic mean of precision and recall. A single number that balances both. Useful when you don't want to optimize for one at the other's expense.

**Generation** — The "G" in RAG. The LLM step that produces an answer using retrieved material as context.

**Golden set** — A small, hand-labeled set of inputs paired with expected behavior. The fixed yardstick against which you measure your system. Foundation of repeatable evaluation.

**Graph RAG** — A RAG variant that builds a knowledge graph of entities and relationships and traverses it before doing vector search. Good for questions that hinge on connections between entities.

**Grounding** — The property of a generation being faithfully derived from the retrieved material rather than the LLM's general knowledge or invention. *Grounded* answers are supported by evidence; *ungrounded* answers may be hallucinations.

**Hallucination** — When an LLM produces confident, fluent, fabricated content not supported by its inputs. RAG reduces hallucination by giving the LLM specific source material to ground its answer in.

**Hybrid search** — Combining keyword retrieval (e.g., BM25) and semantic retrieval (vector search), then merging the results. Catches both exact terms and meaning-based matches.

**LLM-as-judge** — Using one LLM to score the quality of another LLM's output. Useful for subjective dimensions heuristics can't reach. Should be a last resort because it's non-deterministic and adds a fuzzy measuring stick to an already fuzzy system.

**MERT** — An open audio embedding model focused on music understanding. *Music undERstanding Transformer.*

**Metadata filtering** — Applying structured filters (date ranges, categories, tags) *before* vector search to narrow the candidate pool. Cheap, deterministic, and often significantly improves precision.

**MRR (Mean Reciprocal Rank)** — Average of `1 / rank_of_first_relevant_result` across queries. Rewards getting at least one relevant result high up.

**Naive RAG** — The simplest RAG pattern: chunk → embed → vector DB → top-k retrieval → generate. The baseline every system starts from.

**NDCG (Normalized Discounted Cumulative Gain)** — A sophisticated ranking metric that rewards relevant items near the top and partially credits items further down.

**Precision** — Of the items the system flagged as relevant (or retrieved), how many actually were relevant? Higher = fewer false positives.

**Precision@k** — Precision measured over only the top-k retrieved items. "Of my top 10 results, how many were on-target?"

**RAG** — Retrieval-Augmented Generation. A pattern where an LLM answers a query by first retrieving relevant items from your data, then generating an answer using them as context.

**ReAct** — A common agentic-LLM pattern: alternating *Reason* (the LLM thinks about what to do) and *Act* (the LLM calls a tool). Loop continues until the agent decides it's done.

**Recall** — Of all the actually-relevant items in your corpus, how many did the system find? Higher = fewer false negatives.

**Reciprocal Rank Fusion (RRF)** — A simple formula for combining multiple ranked lists into one ranking using *ranks* (not scores). Adds `1 / (k + rank)` from each retriever for each candidate. No score normalization needed.

**Retrieval** — The "R" in RAG. Finding the most relevant items for a query, usually via similarity search over embeddings.

**Similarity search** — The operation of finding the vectors in a database closest to a query vector. The core operation of every vector database.

**SongEval** — A public benchmark dataset for evaluating the aesthetic quality of generated songs. Released around ICASSP 2026. Usable for training Crate's quality classifier.

**Top-k** — The k most-similar items returned by a similarity search. Common values: 5, 10, 20.

**Vector** — A list of numbers. In RAG, "vector" usually means "embedding."

**Vector database** — A storage system optimized for storing many vectors and answering similarity-search queries quickly. Examples: Pinecone, Weaviate, MongoDB Atlas Vector Search, pgvector, Qdrant.
