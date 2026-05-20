# Crate

A content-based discovery engine for AI-generated music — built to surface the good tracks hiding in the flood of AI music nobody hears.

> ~7 million AI-generated tracks are created on Suno every day. About 97% of them never get heard. Collaborative filtering can't help — the tracks are born cold and stay cold. **Crate is the crate you dig through:** it filters out the broken outputs and surfaces the rest by how they actually sound.

## What it does

- **Discovery feed** — a curated stream of AI tracks; broken/noise outputs are already filtered out.
- **"More like this"** — sonic-similarity search over audio embeddings. No interaction data needed.
- **Vibe search** — natural-language queries like *"foggy 2am drive, no vocals"*.
- **Auto-playlists** — coherent sonic clusters with friendly names.
- **Evaluation page** — measured quality, not vibes: filter precision, retrieval precision@10, latency.

## Status

Pre-implementation. The problem statement is locked at `factory/artifacts/PROBLEM_SUMMARY.md`. Subsequent artifacts (`PRD.md`, `LOCKED_DECISIONS.md`, `PLAN.md`) are produced through the Project Factory pipeline.

Study notes on the underlying RAG concepts live in [`notes/`](./notes/README.md).

## Why it exists

A portfolio piece targeted at Suno's open *ML Engineering Manager, Recommendations* role — attacks a real problem Suno publicly cares about, with content-based audio retrieval as the core technique.
