# Issue #147 reproduction

**Issue:** [Resume section detection fails on text with leading whitespace](https://github.com/ascherj/pathreview/issues/147)

## Environment and target

The affected behavior is implemented by `ResumeParser._detect_sections()` in `ingestion/parsers/resume_parser.py`. The method is used for both string/Markdown input and text extracted from PDF files.

## Reproduction

Run the following from the repository root:

```python
from ingestion.parsers.resume_parser import ResumeParser

parser = ResumeParser()
result = parser.parse(
    "\n    John Smith\n"
    "    john@example.com\n\n"
    "    Education:\n"
    "    - B.S. Computer Science\n\n"
    "    Skills: Python\n"
)

print(result.metadata["detected_sections"])
```

## Observed behavior

```text
[]
```

No sections are detected even though the input contains recognizable `Education` and `Skills` headers.

## Expected behavior

The metadata should include both detected sections, regardless of their indentation:

```text
["Education", "Skills"]
```

The exact list order is not currently guaranteed because `_detect_sections()` iterates over a set and returns a de-duplicated list.

## Root-cause evidence

`_detect_sections()` lowercases the text and tests each known section against patterns such as:

```python
rf"^{re.escape(section)}\s*$"
rf"^{re.escape(section)}\s*[:|-]"
rf"\n{re.escape(section)}\s*$"
rf"\n{re.escape(section)}\s*[:|-]"
```

These patterns require the section name to begin immediately at the start of a line or immediately after a newline. In text extracted from PDFs—and in the indented multiline strings used by the related unit tests—spaces or tabs occur before the header, so none of the patterns match.

## Related failing tests

The issue identifies these tests in `tests/unit/test_resume_parser.py`:

- `test_parse_single_column_resume_text`
- `test_parse_resume_no_work_experience`
- `test_detect_sections`

The latter two visibly use indented multiline strings. Their headers therefore reach `_detect_sections()` with leading whitespace and reproduce the same failure.

## Reproduction boundary

This commit documents the failure only. The production regex and existing tests have not been changed yet; the fix will be implemented and validated in Week 9.
