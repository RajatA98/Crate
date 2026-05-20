# 03 — Key Techniques

Deeper notes on the four techniques worth knowing well: BM25, Reciprocal Rank Fusion, metadata pre-filtering, and a brief look at agentic retrieval.

---

## BM25 — keyword search done right

**BM25** (short for *Best Matching 25* — the 25th iteration of the algorithm) is a classic keyword-based retrieval algorithm. It powers Elasticsearch and most "search engine" implementations.

### How it scores

For a given query and document, BM25 produces a relevance score based on three signals:

1. **Term frequency (TF)** — how often query words appear in the document. More appearances = higher score, but with **diminishing returns** (the second mention of "GEICO" matters less than the first).
2. **Inverse document frequency (IDF)** — how *rare* the word is across the whole corpus. Rare words ("ticker", "GEICO") count more than common ones ("the", "company").
3. **Document length normalization** — short documents that mention the term are weighted higher than long documents where the term is buried in noise.

### Why it still matters in 2026

Embeddings are good at meaning, but they can **smooth over exact matches.** A search for `"BRK.A"` might return generic "Berkshire stock" results because the embedding model treated them as similar. BM25 doesn't — it gives massive weight to the *exact* token `BRK.A`.

**Rule of thumb:** If your domain has technical jargon, product codes, ticker symbols, or named entities that matter precisely, BM25 will catch things embeddings miss.

### In Crate

Track titles, manual tags, and (if available) lyrics are perfect BM25 territory. *"songs with 'rain' in the title"* — BM25 nails this; pure semantic search may not.

---

## Reciprocal Rank Fusion (RRF) — combining rankings

**The problem:** You have two retrievers — say BM25 and vector search — each returning a ranked list. They use **different scoring scales** (BM25 scores are unbounded, cosine similarity is between -1 and 1). You can't just add their scores.

**The solution:** Forget the scores entirely. Combine using **ranks** (position in each list).

### The formula

For each candidate document, RRF computes:

$$\text{RRF}(d) = \sum_{r \in \text{retrievers}} \frac{1}{k + \text{rank}_r(d)}$$

- **rank_r(d)** = position of document `d` in retriever `r`'s ranking (1 for the top result, 2 for second, etc.)
- **k** = a constant, typically 60. It dampens the influence of very low ranks. (Lower k = more weight to top results.)
- A document appears in multiple retrievers → its scores add together → it floats to the top.

### Worked example

Retriever A's ranking: [Doc1, Doc2, Doc3]
Retriever B's ranking: [Doc3, Doc2, Doc4]

With k = 60:
- **Doc1**: 1/(60+1) = 0.0164
- **Doc2**: 1/(60+2) + 1/(60+2) = 0.0322 ← appears in both, mid-rank in each
- **Doc3**: 1/(60+3) + 1/(60+1) = 0.0323 ← appears in both, low in A, top in B
- **Doc4**: 1/(60+3) = 0.0159

Final ranking: **Doc3, Doc2, Doc1, Doc4** — Doc3 wins because it's #1 in B *and* appears in A.

### Why it's beautiful

- No score normalization required.
- Works for any number of retrievers (could be 2, 3, 5...).
- Documents that show up in multiple retrievers naturally rise — a built-in "consensus" signal.
- Implementable in ~10 lines of code.

### Weighting

You can multiply each retriever's contribution by a weight to favor one over the other:

$$\text{RRF}(d) = \sum_{r} w_r \cdot \frac{1}{k + \text{rank}_r(d)}$$

The cookbook uses weight pairs like `[0.7, 0.3]` (keyword-heavy) or `[0.3, 0.7]` (semantic-heavy). Default is `[0.5, 0.5]`.

---

## Metadata pre-filtering — the underrated trick

**The idea:** Apply structured filters *before* vector search to shrink the candidate pool.

### Why it's so effective

A vector search over 1 million items returns the top-k closest, but "closest" doesn't mean "relevant" — you might get 10 close matches that are still wrong because they don't match a hard constraint (wrong year, wrong genre, wrong language).

Filter first:
1. **Hard constraints** (filters) trim the corpus to a relevant subset.
2. **Soft preferences** (vector similarity) rank within that subset.

The result is more relevant and often *faster* (vector search on 1,000 items is faster than on 1,000,000).

### Filter types worth knowing

- **Exact match** — `genre = "ambient"`
- **Range** — `bpm BETWEEN 70 AND 100`
- **Set membership** — `key IN ("A minor", "C major")`
- **Boolean flags** — `instrumental = true`
- **Date ranges** — `created_at >= "2024-01-01"`

Most modern vector databases (MongoDB Atlas, Pinecone, Weaviate, pgvector) support filtering natively — you pass filters alongside the query vector and they apply them efficiently.

### Crate application

Every Crate track has rich metadata derived during ingestion: `bpm`, `key`, `energy`, `duration`, `instrumentation_tags`, `vocal_present`. Vibe search for *"calm acoustic, no drums"*:

1. Filter: `energy < 0.4 AND vocal_present = false AND "drums" NOT IN instrumentation_tags`
2. Vector search the remaining tracks for sonic similarity to the implicit query.

---

## Agentic retrieval — a brief tour

**The shift:** From a fixed pipeline ("always do BM25 + vector + RRF") to a flexible one where an LLM decides what to do.

### The ReAct loop

A common agentic pattern:

```
loop:
  Thought: "The user is asking about X, I think I need Y"
  Action: call a tool (search_by_keyword, search_by_meaning, decompose_query, ...)
  Observation: result of the tool
  Thought: "That gave me Z, do I need to keep going?"
  ...until the agent decides it has enough.
```

The LLM is given a few **tools** as available functions, plus their docstrings, and reasons about which to call when.

### Patterns the cookbook highlights

- **Tool selection** — picking the right retrieval method for the query.
- **Query decomposition** — breaking *"how did Berkshire's insurance segment compare to its railroad segment in 2018"* into two sub-queries.
- **Retry / refinement** — if the first retrieval is weak, the agent reformulates and tries again.

### Trade-offs

| Pro | Con |
|---|---|
| Handles complex, multi-faceted queries | Slow (multiple LLM calls per question) |
| Adapts strategy to each query | Expensive |
| Decomposes hard questions naturally | Hard to debug — agent's reasoning can be opaque |
| | Harder to evaluate — non-determinism layered on non-determinism |

### Honest take for Crate

Skip for v1–v3. Revisit only if naive + metadata + hybrid is provably hitting a ceiling on the eval set. Premature agentic = engineering debt with no payoff.

---

## Self-check

1. What are BM25's three signals, and what does each contribute?
2. When does BM25 outperform pure vector search?
3. Why does RRF use *ranks* instead of *scores*?
4. In RRF, what does the constant `k` control?
5. Why is metadata pre-filtering both cheap and effective?
6. What is the ReAct loop, and what makes agentic RAG harder to evaluate?
