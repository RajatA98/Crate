# RAG Study Notes

Notes for learning **Retrieval-Augmented Generation (RAG)** — the foundation of the Crate project. Written to be re-read and revised from later.

## Reading order

1. **[01 — RAG fundamentals](./01-rag-fundamentals.md)** — what RAG is, what problem it solves, the building blocks (embeddings, vector databases, similarity search). **Start here.**
2. **[02 — The RAG ladder](./02-the-rag-ladder.md)** — the five RAG patterns (Naive → Metadata-filtered → Hybrid → Graph → Agentic), and how to choose between them. Based on the [Gauntlet AI RAG Cookbook](https://github.com/Gauntlet-AIDP/rag-cookbook).
3. **[03 — Key techniques](./03-key-techniques.md)** — deeper dives on BM25, hybrid search, Reciprocal Rank Fusion (RRF), and metadata pre-filtering.
4. **[04 — Evaluation](./04-evaluation.md)** — how to test a RAG system. Heuristics + golden sets first, LLM-as-judge later.
5. **[05 — Applying it to Crate](./05-applying-to-crate.md)** — how every RAG concept maps onto the Crate project, with a staged v1 → v3 build path.
6. **[Glossary](./glossary.md)** — key terms in one place.

## If you only have 10 minutes

Read **01 (fundamentals)** + the table at the top of **02 (the ladder)**. That gives you the mental model and the vocabulary. Come back for the rest.

## How to study these

- Each file ends with **self-check questions**. Try to answer them without scrolling up before moving on.
- The **glossary** is your safety net — if a term feels fuzzy, look it up there first.
- Examples are tied to Crate wherever possible so the concepts stick to something concrete you're building.
