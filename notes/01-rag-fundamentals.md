# 01 — RAG Fundamentals

## TL;DR

**RAG = Retrieval-Augmented Generation.** It's a pattern where an LLM (like Claude) answers a question by first **looking things up** in your data, then **using what it found** to write the answer. You're handing the model the relevant material instead of asking it to remember.

---

## The problem RAG solves

An LLM is smart but has two limits:

1. **It doesn't know your specific data.** It wasn't trained on your music corpus, your company's docs, or last week's emails.
2. **If it doesn't know something, it may make it up.** This is called **hallucination** — confident, fluent, wrong.

RAG fixes both by adding a *lookup* step before the LLM speaks. The LLM is no longer guessing from memory — it's working from material you handed it.

> **Analogy:** Asking someone a question from memory vs. handing them the relevant files first and saying "answer using these."

---

## The two halves

RAG is a two-step pattern. The name literally describes the steps:

### 1. **R**etrieval

Find the most relevant items for the query. **This is not keyword search** — it's *meaning-based* search.

### 2. **A**ugmented **G**eneration

Take those retrieved items, put them into the LLM's prompt as context, and ask the LLM to write the answer **using** them.

That's it. The cleverness is in step 1.

---

## The building blocks of retrieval

Three concepts make meaning-based retrieval work:

### Embeddings — "GPS coordinates for meaning"

An **embedding** turns any item (a sentence, a document, a song) into a long list of numbers — typically a few hundred to a few thousand. That list is called a **vector**.

The key property: two items that **mean similar things** get vectors that sit **close together** in number-space.

- "Happy upbeat pop" and "cheerful dance track" → close vectors (even though they share no words).
- "Happy upbeat pop" and "slow funeral dirge" → far-apart vectors.

You produce embeddings by running items through an **embedding model** — a neural network specifically trained to put meaning into number-space. Examples: OpenAI's `text-embedding-3`, CLAP for audio, MERT for music.

> **Why "GPS coordinates"?** Just like GPS gives every location on Earth a precise numeric address, embeddings give every meaning a precise numeric address. Searching becomes "find the addresses closest to mine."

### Vector database — storage built for similarity search

A **vector database** is a storage system that holds millions or billions of these vectors and answers one question very fast:

> *"Given this vector, which of my stored vectors are closest?"*

Normal databases are great at "find me row 42" or "find me everyone named Smith." They're terrible at "find me the 10 vectors closest to this one" — which is exactly the question RAG asks all the time. Vector databases (Pinecone, Weaviate, MongoDB Atlas Vector Search, pgvector) are purpose-built for it.

### Similarity search — the actual lookup

The lookup itself works like this:

1. Take the query (text, audio, whatever).
2. Run it through the **same** embedding model used for the stored items → a query vector.
3. Ask the vector database for the **top-k** (e.g., top 5) closest stored vectors.
4. Return the items those vectors point to.

"Closeness" is usually measured by **cosine similarity** — a number between -1 and 1, where 1 means "exactly the same direction" (very similar) and 0 means "unrelated."

---

## A worked example — text RAG for a support chatbot

User asks: *"How do I reset my password?"*

**Setup (done once, offline):**
- Take every help-doc paragraph in your company → embed each → store in vector DB.

**At query time:**
1. Embed the question → query vector.
2. Vector DB returns the 3 closest help-doc paragraphs.
3. Build the LLM prompt: *"Using the following help docs, answer the user's question. Docs: [paragraph 1, paragraph 2, paragraph 3]. Question: How do I reset my password?"*
4. LLM writes an answer **grounded** in those paragraphs.

Without retrieval → the bot guesses, possibly makes up steps. With retrieval → it answers from your real docs.

---

## Same pattern, different modality — Crate

The exact same machinery, but the "documents" are **songs** and the retrieval works on **how audio sounds** instead of what text means.

| Text RAG | Crate (audio RAG) |
|---|---|
| Documents | Songs |
| Embedding model | CLAP / MERT (audio embedding models) |
| Embedding captures | Meaning of text | How a song sounds (mood, tempo, instrumentation) |
| Query | A text question | A seed track or a vibe phrase |
| Generation step | An answer paragraph | A Suno prompt grounded in the retrieved cluster |

The retrieval engine is *modality-agnostic* — embeddings + vector DB + similarity search work the same for text, audio, images, video, code. Only the embedding model changes.

---

## Why this matters

- RAG is the **most in-demand AI engineering skill in 2026** (per 2026 hiring guides).
- A RAG system over *audio* is harder and rarer than text RAG → stronger portfolio signal.
- The pattern is **composable** — you can keep climbing the ladder (next file) to make it more sophisticated.

---

## Self-check

Answer without scrolling up:

1. What does RAG stand for, and what are the two halves?
2. What is an embedding, in your own words?
3. What problem does a vector database solve that a normal database can't?
4. What's the difference between retrieval done with embeddings and old-fashioned keyword search?
5. In Crate, what plays the role of "documents"?
