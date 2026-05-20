---
name: PROBLEM_SUMMARY
description: Crate — a content-based discovery engine for AI-generated music (no tech decisions)
status: Complete
last_updated: 2026-05-20
---

# Problem Summary — Crate

## The problem

Suno generates roughly 7 million AI-music tracks every day. Industry data shows AI tracks make up about 44% of daily uploads on major streaming services — yet they draw only 1–3% of actual streams. **Around 97% of AI-generated music is never heard by anyone.**

The reason is that the standard music-discovery playbook breaks at this scale. Streaming services use **collaborative filtering** — "people who liked X also liked Y" — which needs each track to accumulate thousands of listens before any pattern emerges. Most AI-generated tracks never get those listens. They're born cold and stay cold forever.

The gap that nothing fills: a way to recommend AI-generated music by **how it actually sounds**, not by who listened to it. Content-based discovery doesn't need interaction history — it works directly from the audio. That's the gap Crate closes.

**Crate is the crate you dig through.** It listens to a corpus of AI-generated tracks, filters out the broken and low-quality ones, and surfaces the good music by sonic similarity, mood, and vibe. The user finds tracks they actually like instead of drowning in undifferentiated output.

## Who it's for

### Primary audience: Suno hiring reviewers

Crate is built as a portfolio piece for a Suno application — specifically targeting the open **ML Engineering Manager, Recommendations** role on Suno's careers board. Reviewers will judge it on:

- product judgment (is this attacking a problem Suno actually has)
- engineering credibility (real audio ML, real retrieval infrastructure, real evaluation)
- execution polish (does the demo feel production-capable)
- discovery quality (do the recommendations feel genuinely good)

### Functional users: anyone curious about AI-generated music

A visitor lands on the app, browses a curated discovery feed, clicks a track that catches their ear, gets sonically similar tracks instantly, can search by vibe phrase, and walks away with a small playlist they'd actually listen to.

### Self-use: the project owner

Used personally to find listenable AI music — closing the "where's the good stuff" gap that exists today.

## The value

- **For the reviewer:** A working demo of audio-ML-powered discovery that maps directly to an open role at the company. End-to-end production thinking — corpus ingestion, embeddings, vector retrieval, quality filtering, evaluation — not an API wrapper.
- **For the visitor:** A genuinely usable music-discovery experience for AI-generated tracks. Find good music, in seconds, without wading through 97% noise.
- **For the owner:** A discovery layer for AI music that doesn't exist anywhere else today.

## Core flow (product level)

1. User lands on a clean discovery feed. The feed has already been filtered — no broken, silent, or hiss-only tracks reach the user.
2. Tracks display with a title, generated metadata (mood, tempo, instrumentation tags), and a play button.
3. User clicks any track → **"more like this"** returns sonically similar tracks instantly.
4. User can search by **vibe phrase** in natural language ("foggy 2am drive, no vocals," "aggressive synth energy").
5. **Auto-generated playlists** group the corpus into coherent sonic clusters with friendly names ("Warm acoustic mornings," "Late-night ambient").
6. Each recommendation shows a **"why this"** line — what it matched on (e.g., "shared mood: hazy; tempo within 80–100 BPM; both instrumental").
7. A visible **evaluation page** reports the system's measured quality: filter precision, retrieval precision@10, recommendation latency. The numbers, not the vibes, prove the system works.

## Constraints (product, not tech)

- **Solo developer, portfolio-feasible timeline.** Every choice has to be buildable by one person to demo quality.
- **Corpus: ~300 tracks generated on Suno's free tier.** Authentic to the project's premise (genuine AI music), small enough to curate by hand, big enough for the discovery experience to feel populated.
- **Polished demo bar.** The standard is "production-feeling prototype," not "weekend hack." UX, copy, error states, and evaluation visibility all need to be credible.
- **Heuristics-first evaluation methodology.** Locked preference: golden sets and deterministic metrics first; LLM-as-judge only after heuristic coverage is exhausted and gated on explicit acknowledgement.
- **Resilient to imperfect data.** Some Suno tracks are noisy, some short, some malformed — the system must filter and surface gracefully, never break.
- **Visual identity:** logo concept is a **crate of records** — leaning into the "crate-digging" metaphor (the craft of unearthing hidden tracks in a record bin).

## Non-goals

- **No music generation in MVP.** Generating new Suno prompts is a stretch feature only, not part of the MVP. The headline is discovery, not creation. This is a deliberate choice to keep the project mapped to the Recommendations role and to avoid any collision with Suno's shipped "My Taste" feature.
- **No collaborative filtering.** "People who liked X also liked Y" is the playbook Crate exists to *replace* — it doesn't work at AI-music scale.
- **No Spotify integration.** No external taste import. Crate stands on its own corpus.
- **No Suno API integration.** Crate doesn't call out to Suno at runtime. The corpus is pre-generated and curated.
- **No user accounts or persistence.** Stateless per visit — same demo principle that worked for the prior pivot.
- **No mobile-first design.** Desktop-focused; remains usable on mobile but not optimized for it.
- **No real-time scale engineering.** Designed for portfolio-demo traffic, not production load.
- **No social or sharing features.** No comments, follows, shared playlists.

## Success criteria (product level)

- A reviewer opens the deployed URL and within 30 seconds sees a curated discovery feed of AI-generated tracks where the broken/noise tracks have already been removed.
- "More like this" returns visibly relevant sonic neighbors — a lo-fi track returns other lo-fi tracks, a synth-heavy track returns other synth-heavy tracks.
- Vibe search ("foggy 2am drive") returns results that match the described mood, not random tracks.
- The evaluation page shows real numbers (filter precision, retrieval precision@10, latency p95) backed by a hand-labeled golden set — not vibes.
- The whole experience feels designed and intentional, not held together with tape. Copy, layout, and error states are considered.
- A reviewer can connect the project to Suno's open ML EM (Recommendations) role in one sentence.

## Open product questions

- **Final name.** "Crate" is the working name and the user has embraced it (crate-of-records logo). Confirming as the final name in PRD unless something stronger emerges.
- **Deployment target.** Where the demo is hosted, and what the URL looks like, gets decided during the architectural phase.
- **Stretch features after MVP ships:**
  - The RAG generation loop (discover a cluster → Claude writes a grounded Suno prompt → user creates their own). Would turn Crate into a true RAG application, but only added if MVP polish has clearance.
  - Hybrid retrieval (BM25 over text metadata + audio embeddings, fused with Reciprocal Rank Fusion). Improves precision on text-shaped queries; adds engineering surface.
  - Optional Spotify "seed" — let a user paste in their Spotify favorites to bias the initial discovery feed.
- **Persistence of the curated corpus.** How and where the ~300 Suno-generated tracks are stored, versioned, and re-loadable. Decided in the architectural phase.

All technical decisions — stack, audio embedding model choice, vector database, hosting, evaluation tooling, deployment — live in subsequent artifacts produced by later phases of the Project Factory pipeline (`PRESEARCH.md`, `LOCKED_DECISIONS.md`, `PLAN.md`).
