# PathReview Module 3 Journal

## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/147

**Issue title:** Resume section detection fails on text with leading whitespace

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
In `ingestion/parsers/resume_parser.py`, `_detect_sections()` looks for resume headers like Experience, Education, and Skills using regex patterns anchored at the very start of a line (`^` or right after `\n`). PDF-extracted text often keeps leading spaces or tabs before those headers, so the patterns never match and `detected_sections` comes back empty even when the sections are clearly present. That breaks downstream review logic that relies on knowing which sections exist. A successful fix should make section detection tolerate leading whitespace (without changing the meaning of the headers), so indented resumes still report the correct sections, and the related unit tests in `tests/unit/test_resume_parser.py` should pass.

**Selection notes ("Is this right for me?"):**
- Scope is clear and localized: one method (`_detect_sections`) and related regex patterns; possibly the same whitespace assumption in `_strip_markdown`.
- Tier 1 / good first issue — realistic for a first contribution to a multi-module repo.
- Reproducible with the provided snippet; failing tests are already named.
- Does not require deep knowledge of RAG, the agent, or the frontend to ship a correct fix.
- Risk of overlap: many classmates claimed #147, so my PR needs a clean, well-tested fix and a clear walkthrough rather than racing to be first.

**Branch name:** fix/147-resume-section-whitespace

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger
