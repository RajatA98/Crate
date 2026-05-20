# 04 — Evaluation: How to Test a RAG System

## TL;DR

RAG outputs aren't right/wrong — they're better/worse. So testing has two layers: **ordinary pass/fail tests** for the plumbing, and **measurement-based evaluation** for the AI quality. The methodology principle: **deterministic heuristics + golden sets first; LLM-as-judge only later, for the cases where heuristics genuinely can't reach.**

---

## The core challenge

Normal software is deterministic — same input, same output, write a test that asserts equality. RAG isn't:

- The retrieval may return *different* but *equally good* top-k results.
- The generation may phrase the answer differently every time.
- There's rarely a single "correct" answer.

So the right question isn't *"is the output correct?"* — it's *"is it good, and how do I prove it with numbers?"*

---

## Two layers of testing

### Layer 1 — the plumbing (deterministic, pass/fail)

The non-AI parts behave normally and get normal tests:

- **Ingestion robustness** — corrupt file, 3-second clip, silent track → handled gracefully, doesn't crash.
- **API contracts** — backend returns the right shape, clean errors on bad input, vector DB returns the requested number of results.
- **Pipeline integrity** — a newly added item actually shows up as retrievable.

Write these as you build, run them on every commit.

### Layer 2 — AI quality (measurement-based)

This is where the cookbook-style **evals** live. You use a **golden set** — a small, hand-curated batch of inputs where you *know* what good behavior looks like — and you measure each component against it.

---

## Methodology principle: heuristics-first, LLM-judge later

**Heuristics and golden sets are deterministic.** Run them a hundred times, get the same result. They give you a **stable regression baseline** — a fixed line that lets you say "did I just make it worse?"

**LLM-as-judge is non-deterministic.** Same answer scored twice by the same judge can drift. If your measuring stick is fuzzy, you can't tell whether a number moved because the system changed or because the judge drifted.

> **Rule:** Build heuristic evals first. Only reach for LLM-judge when you've exhausted heuristic options and a measurement is still needed.
>
> Even then, validate the judge against ~20 human-rated examples before trusting it.

This is what was decided for Crate — and recorded in memory ([eval-methodology-preference](../../.claude/projects/-Users-rajatarora-Projects-Crate/memory/eval-methodology-preference.md)).

---

## The evaluation toolkit

### Golden sets — your fixed yardstick

A **golden set** is a small (~20–200), hand-labeled batch of inputs paired with what good behavior looks like.

For Crate, a few examples:

| Type | Input | Expected behavior |
|---|---|---|
| Retrieval | A lo-fi seed track | Top-10 should be ≥80% lo-fi |
| Vibe search | "aggressive synth energy" | These 5 tracks should appear in top-10 |
| RAG generation | Cluster of 80 BPM acoustic | Generated prompt should mention low-tempo + acoustic |
| Quality filter | A silent track | Filtered out |
| Quality filter | A polished synth-pop song | Passes filter |

A golden set is **work to build** — you label by hand — but it's the foundation. Without it you have no baseline.

### Standard retrieval metrics

For retrieval and recommendation:

- **Precision@k** — of the top-k results, what fraction are actually relevant? *"Of the top 10, 8 were lo-fi → 0.80."*
- **Recall@k** — of all the relevant items in the corpus, what fraction made it into the top-k?
- **Mean Reciprocal Rank (MRR)** — average of `1 / rank_of_first_relevant_result`. Rewards getting at least one good result high up.
- **NDCG (Normalized Discounted Cumulative Gain)** — sophisticated ranking metric that rewards relevant items near the top and partially credits items further down.

For most projects, **precision@k** and **recall@k** are the workhorses.

### Standard classifier metrics

For the quality filter (or any yes/no classifier):

- **Precision** — of the items you said "yes" to, how many were actually yes?
- **Recall** — of all the real "yes" items in the corpus, how many did you catch?
- **F1** — harmonic mean of precision and recall (one number that balances both).
- **Accuracy** — total correct / total. Misleading when classes are imbalanced.

### The closed-loop test (Crate's secret weapon)

For RAG systems specifically, a powerful end-to-end test:

1. User wants something → Crate retrieves a cluster.
2. Crate generates a Suno prompt grounded in that cluster.
3. Actually generate a track on Suno using that prompt.
4. **Embed the new track** and measure its distance from the seed cluster.

If the new track lands close to where the user wanted, the entire pipeline provably worked. This is a *single number* that captures end-to-end quality — the killer metric.

---

## Component-level evaluation plan for Crate

| Component | Method | Type |
|---|---|---|
| Quality filter | Hold-out test set with SongEval labels → precision/recall | Golden set |
| Quality filter (v0) | Heuristic checks: clipping, silence ratio, dynamic range, duration | Pure heuristic |
| Retrieval / "more like this" | Hand-labeled cluster golden set → precision@10 | Golden set + metric |
| Vibe search | Hand-labeled `(query → expected tracks)` pairs → hit rate | Golden set + metric |
| RAG prompt generation — grounding | Check generated prompt against cluster's *measured* features (BPM range, instrument tags, mood) | Heuristic |
| RAG prompt generation — coherence | LLM-as-judge with rubric, validated against ~20 human ratings | LLM-judge *(later)* |
| End-to-end | Closed-loop embedding-distance test | Metric |

**Notice:** almost everything is heuristic or golden-set. LLM-judge sits in *one* spot, for the only dimension (subjective musical coherence) that heuristics genuinely can't measure.

---

## What the cookbook does — and what we'll do differently

The Gauntlet AI RAG Cookbook implements `groundedness.py` as an **LLM-as-judge** (GPT-4o-mini scores "grounded" vs "not grounded"). For text faithfulness — checking whether an answer is supported by retrieved text — that's defensible: text-against-text faithfulness is genuinely hard to capture with heuristics.

But it reinforces the broader lesson: **even respected cookbooks reach for LLM-judge as a first resort.** For Crate, we'll resist that — and only use LLM-judge where heuristics provably cannot answer the question.

---

## Anti-patterns to avoid

- ❌ **Vibes-only evaluation.** "It feels good." This is not measurement.
- ❌ **Eyeballing 5 examples.** Too small. Numbers drift. You'll fool yourself.
- ❌ **Eval the whole system as one number.** You won't know which component broke. Evaluate components separately *and* end-to-end.
- ❌ **LLM-judge as first move.** You haven't earned the right yet — establish heuristic baseline first.
- ❌ **No regression suite.** If you can't re-run the same evals later and compare, you can't tell whether changes helped or hurt.

---

## Self-check

1. Why can't you use ordinary equality assertions to test a RAG system's output?
2. What is a golden set, and what is it for?
3. What does precision@10 measure?
4. Why is heuristic-first preferred over LLM-as-judge?
5. When is LLM-as-judge actually a defensible choice?
6. Describe the closed-loop test for Crate in one sentence.
