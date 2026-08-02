## Solution plan

**Issue:** [Resume section detection fails on text with leading whitespace — #147](https://github.com/ascherj/pathreview/issues/147)

### Understand

`ResumeParser._detect_sections()` in `ingestion/parsers/resume_parser.py` attempts to recognize common resume headers by matching each value in `SECTION_HEADERS` against several regular expressions. Those expressions anchor the header directly to the beginning of a line (`^`) or directly after a newline (`\n`) and do not permit spaces or tabs before the header.

PDF extraction commonly preserves visual indentation. The unit tests also pass indented multiline strings. As a result, a line such as `    Education:` does not match a pattern that effectively expects `Education:` at column zero. The parser still returns a valid `ParseResult`, but `result.metadata["detected_sections"]` is empty or incomplete.

Expected behavior: known section headers should be detected when they begin a logical line, even if that line has leading horizontal whitespace. Detection should remain case-insensitive, should continue supporting optional separators such as `:`, `|`, and `-`, and should not match section words embedded inside ordinary prose.

Actual behavior: leading spaces or tabs prevent otherwise valid headers from being detected.

### Map

Files and code involved:

- `ingestion/parsers/resume_parser.py`
  - `SECTION_HEADERS`: defines the supported section names.
  - `ResumeParser.parse()`: routes string content to `_parse_markdown()` and bytes to `_parse_pdf()`.
  - `ResumeParser._parse_pdf()`: extracts PDF text and passes it to `_detect_sections()`.
  - `ResumeParser._parse_markdown()`: strips Markdown and passes the resulting text to `_detect_sections()`.
  - `ResumeParser._detect_sections()`: contains the faulty section-header regex patterns and is the primary production file to change.
- `tests/unit/test_resume_parser.py`
  - `test_parse_single_column_resume_text`
  - `test_parse_resume_no_work_experience`
  - `test_detect_sections`
  - This file should also receive focused regression coverage for spaces and tabs before section headers.
- `REPRODUCTION.md`
  - Records the confirmed failing input and root-cause evidence for Week 8.
- `JOURNAL.md`
  - Records the reproduction commit and this plan.

I do not currently expect route, API, model, frontend, or database files to change because the failure is isolated to parser text matching.

### Plan

1. Simplify the section matching in `ResumeParser._detect_sections()` so each known header is matched at the start of a logical line with optional leading spaces or tabs. Use multiline matching rather than maintaining redundant `^...` and `\n...` variants.
2. Preserve the existing accepted header forms: a header alone on a line and a header followed by optional whitespace plus `:`, `|`, or `-`.
3. Add or strengthen focused tests in `tests/unit/test_resume_parser.py` that verify detection with four-space indentation and tab indentation, including `Education:` and an inline-content header such as `Skills: Python`.
4. Run the three issue-related unit tests first, then run the complete `tests/unit/test_resume_parser.py` file to catch regressions in Markdown and PDF parsing behavior.
5. Run the broader unit test suite if the local environment supports it, review the final diff to ensure the change remains limited to section detection, and document any unresolved ordering behavior separately rather than expanding the scope of issue #147.

### Inputs & outputs

**Input:** Resume text supplied as a Python string or extracted from PDF bytes. Relevant lines may contain:

- no indentation: `Education:`
- space indentation: `    Education:`
- tab indentation: `\tEducation:`
- a separator and same-line content: `    Skills: Python`
- mixed capitalization: `    WORK EXPERIENCE:`

**Output:** A `ParseResult` whose `metadata["detected_sections"]` contains the title-cased names of recognized sections. For the reported reproduction, the result should include `Education` and `Skills` rather than an empty list. The parser's extracted `text`, `source_type`, and `page_count` behavior should not otherwise change.

### Risks & unknowns

- A regex that uses unrestricted `\s*` after a multiline anchor can consume newline characters, not just indentation. To avoid crossing line boundaries unexpectedly, the implementation should prefer horizontal whitespace such as `[ \t]*` before the section name.
- Making the pattern too permissive could identify words inside bullets or prose as headers. The match must remain anchored to the beginning of a logical line and require either end-of-line or one of the supported separators after the known section name.
- Some section names overlap, such as `experience`, `professional experience`, and `work experience`. The current implementation loops through each known value independently, so the fix should be checked for duplicate or surprising detections without broadening the issue into a redesign.
- `SECTION_HEADERS` is a set, and `_detect_sections()` returns `list(set(detected))`, so output order is nondeterministic. Issue #147 does not request deterministic ordering; tests should compare membership rather than exact list order unless maintainers indicate otherwise.
- `_strip_markdown()` currently removes Markdown heading markers only when `#` is at column zero. That may be a related whitespace issue, but changing it is outside the confirmed root cause unless testing shows it is required for the named failures.

### Edge cases

The fix should handle gracefully:

- section headers with no leading whitespace;
- headers preceded by multiple spaces;
- headers preceded by one or more tabs;
- uppercase, lowercase, and mixed-case headers;
- headers with a trailing colon, pipe, or hyphen;
- headers followed by same-line content, such as `Skills: Python`;
- headers that appear at the very beginning of the entire document;
- headers that appear after one or more newline characters;
- resumes with only some known sections and no work-experience section;
- empty text or text containing no recognized headers, which should still return an empty list;
- ordinary sentences containing words like “experience” or “skills” away from the start of a line, which should not be classified as section headers.

This document will be updated in Week 9 if implementation or testing reveals additional affected files or constraints.
