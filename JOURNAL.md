## Week 7 — Issue selection

**Issue link:** 
https://github.com/ascherj/pathreview/issues/36

**Issue title:** 
Architecture doc doesn't explain the hybrid retrieval scoring formula

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

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