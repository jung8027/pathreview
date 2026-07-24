## Week 7 — Issue selection

**Issue link:** 
https://github.com/ascherj/pathreview/issues/36

**Issue title:** 
Architecture doc doesn't explain the hybrid retrieval scoring formula

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Why this issue / tier fit:**
I'm still getting oriented in the pathreview codebase, so I picked a Tier 1 issue on purpose rather than jumping into scoring or retrieval logic I don't fully understand yet. The scope is only limited to docs-only (`docs/ARCHITECTURE.md`). This does not require any editing of the project code base so it is low-risk to get wrong. This forces me to actually read and understand the hybrid retrieval scoring code well enough to explain it which is a good on-ramp before I try to modify that code directly in a later issue. The deliverable is to document the formula, default weights, and one worked example into ARCHITECTURE.md. This is a good match for my current comfort level with the codebase.

**Problem summary:**
The Issue
The docs/ARCHITECTURE.md file mentions that the hybrid retrieval system blends vector and keyword scores, but it fails to explain how this blending actually works.

What's Missing
The documentation currently lacks the specific mathematical formula used for the scoring logic, as well as the default weights assigned to the vector and keyword components.

What a Successful Fix Accomplishes
A successful update will provide developers with a clear, dedicated section explaining the exact scoring logic, complete with a concrete, step-by-step example. This will eliminate ambiguity, detail the default configuration, and make the hybrid search behavior fully transparent and reproducible.

**Branch name:** 
docs/36-Architecture-doc-doesn't-explain-the-hybrid-retrieval-scoring-formula

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [link to commit documenting the reproduced issue]

**Reproduction summary:**
[1–2 sentences: How did you reproduce the issue? What did you observe?]

**PLAN.md link:** [link to PLAN.md in your fork]

**Walkthrough video (recommended):** [link to your Loom video, ≤2 min — recommended, not graded]

**Blockers or open questions:**
[Anything you're still uncertain about going into Week 9, or leave blank]

**Scoring logic notes (research for the fix):**

`docs/ARCHITECTURE.md` currently says hybrid retrieval "fetches relevant context from the user's ingested documents" via "vector similarity + BM25 keyword" search, but stops there — it never states the blending formula or the default weights. This is the gap issue #36 asks to close. Below is what I found reading `rag/retriever/hybrid.py`, to carry over into the ARCHITECTURE.md update.

*The formula* (`HybridRetriever.retrieve`, [hybrid.py:78-81](rag/retriever/hybrid.py#L78-L81)):

```
blended_score = vector_weight * normalized_vector_score + keyword_weight * normalized_keyword_score
```

Each side is normalized to 0–1 before blending by dividing by the max score seen in that result set ([hybrid.py:58-59](rag/retriever/hybrid.py#L58-L59)):
- `normalized_vector_score = vector_score / max(vector_scores)`
- `normalized_keyword_score = bm25_score / max(bm25_scores)`

If a chunk was only returned by one of the two searches, the other side's score is treated as 0 rather than excluded.

*Default weights* ([hybrid.py:13-14](rag/retriever/hybrid.py#L13-L14)): `vector_weight=0.7`, `keyword_weight=0.3` — vector similarity dominates the ranking by default, keyword match is a secondary signal.

*Worked example:* query returns three candidate chunks:

| Chunk | Vector score | Normalized vector | BM25 score | Normalized keyword | Blended (0.7 / 0.3) |
|---|---|---|---|---|---|
| A (in both) | 0.90 | 0.90/0.90 = 1.00 | 8.0 | 8.0/8.0 = 1.00 | 0.7×1.00 + 0.3×1.00 = **1.00** |
| B (vector only) | 0.60 | 0.60/0.90 = 0.67 | — | 0 | 0.7×0.67 + 0.3×0 = **0.47** |
| C (keyword only) | — | 0 | 4.0 | 4.0/8.0 = 0.50 | 0.7×0 + 0.3×0.50 = **0.15** |

With the default `min_score=0.3` ([hybrid.py:29](rag/retriever/hybrid.py#L29)), chunk C would be filtered out entirely — it only cleared the bar because keyword_weight is small, which illustrates why the default weighting favors vector-only or both-matched chunks over keyword-only matches.

Note for the fix: `_get_all_chunks` fetches every chunk in the collection but its result is never passed into `keyword_searcher.index()` before `.search()` is called, so keyword scores may currently be empty at runtime unless the searcher was indexed elsewhere beforehand. Worth flagging separately since it's a behavior bug, not a docs gap — but it explains why the "keyword" half of hybrid search may look inactive when testing manually.