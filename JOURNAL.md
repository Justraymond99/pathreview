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

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/Justraymond99/pathreview/commit/2982c4cc9c83c000af32c08a353e02ca2c0fd819

**Reproduction summary:**
I reproduced issue #147 by parsing resume text whose `Education:` and `Skills:` headers have four leading spaces. `_detect_sections()` returned an empty list because its regular expressions require the section name to appear immediately at the start of a line or immediately after a newline.

**PLAN.md link:** https://github.com/Justraymond99/pathreview/blob/fix/147-resume-section-whitespace/PLAN.md

**Walkthrough video (recommended):** Not recorded yet.

**Blockers or open questions:**
The main implementation risk is allowing indentation without making the regex so permissive that it matches section words inside ordinary prose. I also need to avoid using unrestricted `\s*` before headers because it may consume newline characters; `[ \t]*` is the safer candidate for horizontal indentation. Output ordering is currently nondeterministic because `SECTION_HEADERS` and the final de-duplication both use sets, but deterministic ordering appears outside the scope of issue #147.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Implemented the `_detect_sections()` fix using a single multiline regular expression that allows leading spaces or tabs while keeping section names anchored to the beginning of a logical line. Added focused regression tests for space-indented and tab-indented headers, same-line section content such as `Skills: Python`, and a negative prose case. Opened a draft PR in my fork so the diff can be reviewed while final validation is completed.

**Next steps:**
Run the focused resume parser tests, then `make test-unit` and `make check`. Document any pre-existing failures, request peer or mentor feedback on the draft PR, address relevant feedback, open the final PR against the upstream `pathreview` repository, and complete Check-in 2 with the submitted PR link.

**Blockers:**
The connected GitHub integration can update my fork but cannot open the cross-repository PR against `ascherj/pathreview`, so that final upstream PR step may need to be completed manually in GitHub after validation.

---

### Check-in 2 (end of week)

**PR link:** [link to your submitted upstream pull request]

**Branch:** fix/147-resume-section-whitespace

**What you built:**
Updated `ResumeParser._detect_sections()` to detect known resume headers after optional leading spaces or tabs using one multiline, case-insensitive regex per section. The match remains anchored to the beginning of each logical line and still accepts bare headers or the existing `:`, `|`, and `-` separators without matching section words embedded in ordinary prose.

**Tests added or updated:**
Updated `tests/unit/test_resume_parser.py` with regression tests for four-space indentation, tab indentation, same-line content such as `Skills: Python`, and a negative test ensuring ordinary prose containing words such as “skills” and “experience” is not treated as a section header.

**Self-review confirmation:** [ ] make check passes  [ ] make test-unit passes

**Draft PR feedback received from:** [name or Slack handle, or "none"]
