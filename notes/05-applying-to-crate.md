# 05 — Applying RAG to Crate

## TL;DR

Crate is a **RAG system over Suno-generated music**. The user searches and discovers tracks by sonic similarity, vibe, and tags; when they find something they like, Crate uses RAG to write a grounded Suno prompt so they can create their own version. The build follows the cookbook's progression: **v1 Naive → v2 Metadata-filtered → v3 Hybrid**.

---

## The one-sentence pitch

> **Crate helps you find the good music hiding in the flood of AI-generated songs — and create your own from what you find.**

---

## Why RAG fits Crate

The problem Crate attacks:
- ~7 million AI-generated tracks created per day.
- ~97% of them are never heard by anyone.
- Standard music recommendation (collaborative filtering — "people who liked X also liked Y") **breaks at this scale** because almost no track has enough listeners to generate the patterns it needs.

The fix: **content-based retrieval** — recommend by how a song *actually sounds*, not by who listened to it. That's exactly retrieval done with embeddings — the heart of RAG. And once we have retrieval working, layering a **generation step** on top closes the loop into a creation tool.

---

## How each RAG concept maps to Crate

| RAG concept | Crate equivalent |
|---|---|
| Documents | Suno-generated tracks |
| Embedding model | CLAP or MERT (audio embedding models) |
| Embedding captures | How a track sounds — mood, tempo, instrumentation, energy |
| Vector database | Stores audio embeddings + track metadata |
| Query | A seed track, a vibe phrase, or a cluster the user liked |
| Retrieval | Top-k sonically similar tracks |
| Augmentation | Audio features extracted from the retrieved cluster |
| Generation | Claude writes a Suno-ready prompt grounded in those features |
| Closed loop | The new Suno track re-enters the corpus, becomes retrievable |

---

## Staged build path — climbing the ladder

### v1 — Naive RAG (MVP)

The minimum viable demo, end to end.

**Pieces:**
- Ingestion pipeline: take a corpus of tracks → run through CLAP/MERT → store embeddings in vector DB.
- Quality filter v0: deterministic heuristics (clipping, silence, duration, dynamic range) — filter junk *before* it enters the corpus.
- API: "more like this" — accept a seed track, return top-k nearest by embedding.
- Frontend: a discovery feed, a track-detail page with a "more like this" button.
- Evals: golden set of hand-labeled clusters, precision@10.

**This alone beats TasteSync.** Real audio handling, real ML pipeline, deployable, measurable.

### v2 — Add metadata filtering

Once v1 works, extract richer metadata during ingestion and use it to pre-filter.

**Pieces added:**
- Audio analysis during ingestion: BPM, key, energy, vocal presence, duration, instrumentation tags.
- Filter UI: tempo slider, key picker, "instrumental only" toggle, energy range.
- Retrieval API: applies filters *before* the vector search.
- Evals: a/b precision@10 with and without filters (should improve).

**Why this stage:** Cheap engineering, big precision lift. Filters are deterministic — they slot in cleanly without disturbing v1.

### v3 — Hybrid search + RAG creation loop

This is the version a Suno reviewer notices. Two upgrades together:

**Hybrid retrieval (cookbook step 3):**
- Add BM25 over text fields: title, user-supplied prompt, manual tags, lyrics (if available).
- Combine BM25 ranking + audio-embedding ranking with **Reciprocal Rank Fusion (RRF)**.
- Query like *"songs about midnight rain that sound like Bon Iver"* — finally works because both signals contribute.

**RAG creation loop (the showpiece):**
- User finds a track / cluster they like.
- Crate retrieves the cluster → extracts shared audio features (avg BPM, common instruments, dominant mood).
- Claude receives those *measured* features as context and writes a grounded Suno prompt.
- Output: prompt + reference tracks it was built from + a "why this" line.
- Optional: closed-loop test — actually run the prompt on Suno, embed the result, measure distance to seed cluster.

**Evals for this stage:**
- Hybrid precision@10 vs. naive precision@10.
- Grounding heuristic: does the generated prompt's stated tempo fall within the cluster's measured BPM range? Do instruments named match the audio tags?
- Closed-loop embedding distance.

### v4+ — Optional stretch

Only after v3 is solid and the evals plateau:
- **Agentic layer** — let an LLM choose between retrieval methods per query.
- **Spotify seeding** — bring back TasteSync's idea as *one optional input* (Spotify favorites seed the discovery), not the headline.
- **Graph layer** — model artist/scene/genre relationships.

---

## What this looks like to a hiring reviewer

A Suno reviewer scanning the repo and the demo will see, in roughly this order:

1. **Headline that maps to an open Suno role.** Crate is a *discovery* engine — Suno has an open "ML Engineering Manager, Recommendations" role.
2. **Real audio ML, not a wrapper.** Audio embeddings, vector search, an actual ingestion pipeline.
3. **Production-grade vocabulary in the README.** "Hybrid retrieval with Reciprocal Rank Fusion over CLAP audio embeddings and lyric BM25, with metadata pre-filtering."
4. **Evidence of evaluation.** A page with numbers — filter precision, retrieval precision@10, closed-loop distance.
5. **A genuine RAG application** — text + audio retrieval feeding grounded generation. Few candidates ship anything beyond text RAG.

The collision with Suno's shipped "My Taste" feature (which sank TasteSync) is sidestepped entirely: My Taste is in-app passive personalization; Crate is content-based discovery + retrieval-augmented creation. Different problem, different solution.

---

## The decisions captured so far

- **Pivoted from TasteSync to Crate.** See [crate-project-pivot](../../.claude/projects/-Users-rajatarora-Projects-Crate/memory/crate-project-pivot.md).
- **Eval methodology: heuristics + golden sets first, LLM-judge later.** See [eval-methodology-preference](../../.claude/projects/-Users-rajatarora-Projects-Crate/memory/eval-methodology-preference.md).
- **Discovery-first framing.** The headline is *discovery*; the RAG generation loop is dropped from MVP and held as a stretch.
- **Project name:** Crate (folder, repo, and visual identity all locked).

Next step: continue the Project Factory pipeline from `/prd` onward — `factory/artifacts/PROBLEM_SUMMARY.md` is current, `ARCHITECTURE_DIRECTION.md` was removed (subsequent artifacts will be produced during the `/decide` and `/plan` phases).

---

## Self-check

1. Why does collaborative filtering (Spotify-style "people who liked X also liked Y") break for Crate?
2. What plays the role of "documents" in Crate, and what plays the role of "embeddings"?
3. What does v1 (Naive RAG) include for Crate?
4. What's added in v2, and what's added in v3?
5. What makes the v3 RAG creation loop different from a thin LLM wrapper?
6. Why is "discovery-first" framing non-negotiable for this project?
