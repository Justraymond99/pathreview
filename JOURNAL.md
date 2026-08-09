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
Validate the issue-specific parser behavior, document any pre-existing failures, request peer or mentor feedback if available, finalize the PR, and complete Check-in 2 with the submitted PR link.

**Blockers:**
The connected GitHub integration can update my fork but GitHub returned `403 Resource not accessible by integration` when I attempted to open the cross-repository PR against `ascherj/pathreview`. I therefore finalized the available PR on my fork before the deadline and documented the limitation in the PR description.

---

### Check-in 2 (end of week)

**PR link:** https://github.com/Justraymond99/pathreview/pull/1

**Branch:** fix/147-resume-section-whitespace

**What you built:**
Updated `ResumeParser._detect_sections()` to detect known resume headers after optional leading spaces or tabs using one multiline, case-insensitive regex per section. The match remains anchored to the beginning of each logical line and still accepts bare headers or the existing `:`, `|`, and `-` separators without matching section words embedded in ordinary prose.

**Tests added or updated:**
Updated `tests/unit/test_resume_parser.py` with regression tests for four-space indentation, tab indentation, same-line content such as `Skills: Python`, and a negative test ensuring ordinary prose containing words such as “skills” and “experience” is not treated as a section header. Targeted A/B validation using the same resume-parser cases showed the upstream baseline at 6 passing / 7 failing and this branch at 11 passing / 2 failing. The two remaining failures are pre-existing `_strip_markdown()` failures (`test_parse_markdown_resume` and `test_strip_markdown_syntax`) and are unchanged by this scoped fix.

**Self-review confirmation:** [ ] make check passes  [ ] make test-unit passes

Full Makefile validation could not be executed from the connected environment before the deadline. The issue-specific parser behavior and touched tests were validated independently, and no new failures were observed in the targeted before/after comparison.

**Draft PR feedback received from:** none

## Week 10 — Iteration & reflection

### Reviewer feedback

**Feedback received:** [ ] Yes  [x] No — still awaiting review

**Summary of feedback:**
No reviewer feedback was provided for Summer 2026. I reviewed the submitted PR and the Week 9 validation notes again to make sure the implementation, tests, and documented limitations were clear enough for a maintainer or grader to follow without additional context.

**How you responded:**
No code-review response was required because no reviewer feedback was provided. I left the implementation scoped to issue #147 rather than expanding it into the related `_strip_markdown()` whitespace behavior that surfaced during testing.

---

### Reflection

**What was harder than you expected?**
The hardest part was not writing the regex itself; it was proving that a small change was actually the correct change and did not create new behavior elsewhere. The initial bug looked simple, but once I traced the parser I had to think about multiline matching, spaces versus tabs, separators such as `:`, `|`, and `-`, false positives in ordinary prose, and the fact that `\s*` can also consume newlines. Testing also exposed unrelated failures in `_strip_markdown()` and broader repository checks, so I had to separate failures caused by my work from failures that already existed. The contribution workflow was also more involved than I expected: branch conventions, reproduction notes, PLAN.md, tests, PR documentation, and keeping the fix narrow all mattered in addition to the code.

**What did you learn about working in a large codebase?**
I learned that contributing to someone else's codebase is much more about understanding boundaries than immediately changing code. In my own projects I can redesign a function if I dislike it, but here I needed to understand what `_detect_sections()` promised to do, which callers depended on it, what existing tests expected, and what issue #147 actually asked for. I also learned that a failing repository-wide test suite does not automatically mean my change is wrong. The useful question is whether the failure existed before my change and whether the changed behavior is covered by focused regression tests. That required comparing the before-and-after behavior instead of treating every red test as part of my issue.

**How did AI tools help — and where did they fall short?**
AI was most useful for navigating the unfamiliar repository, tracing the flow from `parse()` into `_parse_pdf()` / `_parse_markdown()` and then `_detect_sections()`, identifying the regex root cause, comparing possible regex approaches, and helping generate focused test cases. It also helped me organize the reproduction, PLAN.md, PR description, and journal so the reasoning behind the change was documented instead of only presenting a patch. Where AI fell short was environment and permissions. It could reason about the code and validate targeted behavior, but it could not replace every real repository action: the connected environment could not run the full local Makefile workflow before the deadline, and the GitHub integration could not create the cross-repository upstream PR because of permission restrictions. I still had to make judgment calls about scope and be explicit about what was and was not actually verified rather than claiming tools had done more than they had.

**What would you do differently if you started over?**
I would capture the repository baseline earlier and run the exact required commands before changing anything. That would make pre-existing failures easier to document and would reduce uncertainty at submission time. I would also open the upstream draft PR as soon as the reproduction and plan were complete instead of leaving repository permissions and final PR mechanics until late in the process. On the implementation side, I would keep the same narrow approach: fix `_detect_sections()` first, add positive and negative regression cases, and treat the related `_strip_markdown()` behavior as a separate issue unless the maintainer explicitly wanted the scope expanded.

**What are you most proud of from this module?**
I am most proud that I moved from simply finding a failing test to being able to explain the actual root cause and defend the shape of the fix. The final change is small, but it is intentionally small: it allows horizontal indentation, stays anchored to logical line starts, preserves the existing section formats, and includes tests that guard both the bug and the risk of over-matching. The biggest takeaway for me is that a strong contribution is not measured by how many lines of code changed; it is measured by how clearly the problem is understood, how safely the behavior is changed, and how well the evidence is documented.
