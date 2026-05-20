# 02 — The RAG Ladder

## TL;DR

RAG is not a single technique — it's a **ladder** of increasingly sophisticated patterns. The right move is to **start at the bottom and only climb when the simpler version isn't enough.** The [Gauntlet AI RAG Cookbook](https://github.com/Gauntlet-AIDP/rag-cookbook) lays out five rungs.

---

## The five patterns

| Step | Pattern | One-line summary | When to use |
|---|---|---|---|
| 1 | **Naive RAG** | Chunk → embed → vector DB → top-k | Always — the baseline |
| 2 | **Metadata filtering** | Pre-filter by category before vector search | When items have structured attributes |
| 3 | **Hybrid search** | Combine keyword + semantic search, fuse rankings | When exact terms matter alongside meaning |
| 4 | **Graph RAG** | Build a knowledge graph, traverse relationships | When relationships drive the answer |
| 5 | **Agentic RAG** | An LLM decides whether/how to retrieve and may retry | Complex multi-step queries |

---

## Step 1 — Naive RAG (the foundation)

**The pattern:**
1. **Chunk** documents into manageable pieces (e.g., 500 tokens each).
2. **Embed** each chunk.
3. **Store** the embeddings in a vector database.
4. At query time: embed the query → fetch the **top-k** nearest chunks → feed to the LLM.

**That's it.** No bells, no whistles. Every RAG system starts here.

**Why start here:** It's the baseline against which every later improvement is measured. If your naive RAG already scores well on your evals, you might not need anything fancier.

**Limitation:** Pure semantic search can miss exact matches. A query for "GEICO 2020 earnings" might miss a chunk that uses "GEICO" but doesn't paraphrase "earnings" — semantic search doesn't always weight exact terms.

---

## Step 2 — Metadata filtering (targeted retrieval)

**The pattern:** Before doing the expensive vector search, narrow the candidate pool using **structured filters** on the items' metadata.

**Example:** Instead of searching across all 20 years of Buffett letters, filter to `year >= 2018 AND topic = "insurance"` *first*, then run vector search on what remains.

**Why this is cheap and powerful:**
- Smaller candidate set → top-k results are more relevant.
- Less noise reaching the LLM → cleaner generation.
- Filters are deterministic and fast — they cost almost nothing.

**Requirement:** Your items must *have* structured metadata. If you stored only the text and the embedding, you can't filter — you need fields like `date`, `genre`, `bpm`, `tags`.

**Crate version:** Filter tracks by `BPM 70–100, instrumental = true, key = A minor` *before* the audio vector search. The vibe search "calm acoustic, no drums" becomes:
1. Filter for instrumentals with low energy and BPM under 100.
2. Vector-search audio embeddings within that subset.

Much higher precision than embeddings alone.

---

## Step 3 — Hybrid search (best of both worlds)

**The pattern:** Run **two** retrievers in parallel, then **fuse** their results into one ranking.
- **BM25** — a classic keyword-matching algorithm (covered in detail in [03 — Key techniques](./03-key-techniques.md)). Catches *exact* matches — ticker symbols, names, technical terms.
- **Vector search** — catches *meaning* matches — paraphrases, vibes, conceptual similarity.

The two rankings get blended using **Reciprocal Rank Fusion (RRF)** — a simple formula that combines rankings without needing the scores to be on the same scale.

**Why it's better:** Each retriever catches what the other misses. Semantic search alone misses exact terms; keyword search alone misses paraphrases. Combined, they cover both.

**Tuning knob:** A weighting parameter controls the balance. The cookbook uses `[0.5, 0.5]` (equal) by default, `[0.7, 0.3]` to favor keywords, `[0.3, 0.7]` to favor semantics.

**Crate version:** Crate has *two* signals per track:
- **Audio embedding** (semantic — captures how it sounds).
- **Text metadata** — title, user prompt, manual tags, possibly lyrics (keyword — BM25 territory).

Hybrid lets Crate answer queries like *"songs about midnight rain that sound like Bon Iver"* — semantic alone misses "midnight rain" in lyrics; keyword alone misses sonic similarity. Hybrid catches both.

---

## Step 4 — Graph RAG (relationship-aware)

**The pattern:** Extract **entities** (people, places, organizations) and the **relationships** between them from your documents. Build a knowledge graph. At query time, traverse the graph *before* doing vector search.

**Why:** Some questions need *connected* information. "Which companies did Buffett acquire after divesting from textiles, and what were their CEOs' backgrounds?" requires hopping from one entity to another — graphs do this naturally.

**Cost:** Building and maintaining a knowledge graph is significant engineering work. Worth it only when relationships clearly drive the answers.

**Crate take:** Probably overkill. Crate's value lives in *audio* similarity, not relationships between artists/genres. Could be a future flourish (e.g., "artists in the same scene as X") but not v1, v2, or v3.

---

## Step 5 — Agentic RAG (dynamic decision-making)

**The pattern:** Instead of a fixed retrieval pipeline, give an LLM a few retrieval methods as **tools** and let it decide:
- *Whether* to retrieve at all.
- *Which* method to use.
- *Whether to retry* with a different query if the first try was weak.
- *Whether to decompose* a complex question into sub-questions.

Often follows the **ReAct** pattern — Reason, then Act — where the LLM alternates between thinking and tool-calling.

**When it shines:** Complex, multi-faceted questions where a fixed pipeline would always miss something.

**Cost:** Slower, more expensive, harder to debug, and harder to evaluate. The agent's decisions add a layer of non-determinism on top of an already non-deterministic LLM.

**Crate take:** Potential stretch goal — an agent could decide *"search by mood, by similar track, or by lyrics?"* — but premature before steps 1–3 are solid.

---

## The big lesson: complexity vs. benefit

The cookbook's philosophy in one sentence:

> **Start simple. Add sophistication only when measurement proves you need it.**

Each climb up the ladder adds engineering cost, latency, and debugging surface. You don't earn that cost back unless the simpler version is *demonstrably* failing — which means you must have **evals** in place from the start (next file).

---

## Self-check

1. List the five RAG patterns in order, with a one-line summary of each.
2. What is the prerequisite for using metadata filtering?
3. In hybrid search, what does BM25 catch that semantic search misses, and vice versa?
4. What kind of question is Graph RAG good at?
5. Why is "start at step 1" the right default, even if step 5 is more impressive?
