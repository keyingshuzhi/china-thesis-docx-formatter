---
name: academic-paper-formatter
description: Extract rules from a Chinese university Word template, then format and validate domestic undergraduate, master's, and doctoral thesis DOCX files without school-specific hard-coding.
metadata:
  short-description: Template-driven Chinese thesis DOCX formatter
---

# Academic Paper Formatter

Use this skill for Chinese domestic undergraduate, master's, and doctoral
thesis `.docx` files. The school-provided template is the formatting authority:
analyse it first, generate a rules JSON, then format a separate working copy of
the thesis. Do not replace this flow with an assumed university standard.

Choose a degree level only to surface a manual structural checklist. It never
adds a required section or overwrites a template-derived format:

```bash
python scripts/analyze_docx.py school-template.docx --degree master --output school-rules.json
```

## Required workflow

1. Keep the original template and thesis untouched. Work on copies only.
2. Generate rules from the school's template:

   ```bash
   python scripts/analyze_docx.py school-template.docx --output school-rules.json
   ```

3. Inspect `school-rules.json`. Confirm ambiguous roles such as cover title,
   Chinese/English abstracts and keywords, declaration page, contents, heading
   levels, captions, references, acknowledgements, appendices, and degree-period
   research achievements against the template. These components vary by school
   and degree; edit only confirmed values and record unresolved warnings.
4. Validate the generated rules:

   ```bash
   python scripts/validate_format.py school-rules.json
   ```

5. Format a copied thesis and then validate it:

   ```bash
   python scripts/format_document.py thesis-source.docx thesis-formatted.docx --rules school-rules.json
python scripts/validate_format.py school-rules.json --document thesis-formatted.docx
```

For a review-only domestic degree structure check, add `--degree undergraduate`,
`--degree master`, or `--degree doctoral`. Use `--strict-degree` only after the
school's written requirements confirm that every flagged component is mandatory.

6. Render the resulting DOCX and visually inspect every page before delivery.
   Check page geometry, Chinese font rendering, page breaks, table clipping,
   captions, headers/footers, and reference wrapping.

## Advanced workflow

### Content completion

First locate placeholders. Draft proposed text in a separate review file or in
conversation; do not silently generate or insert academic claims. After the
user approves the exact mapping, apply it to a new DOCX copy:

```bash
python scripts/complete_content.py thesis.docx --report placeholders.json
python scripts/complete_content.py thesis.docx --replacements approved-content.json --output thesis-completed.docx
```

`approved-content.json` is an object mapping the exact placeholder text to the
approved replacement. The tool preserves paragraph structure and reports
placeholders split across Word runs instead of replacing them unsafely.

### Citations and references

Audit numeric citations against the existing bibliography before changing it:

```bash
python scripts/manage_references.py thesis.docx --rules school-rules.json --report references-audit.json
```

The report finds missing entries, uncited entries, and duplicate labels. It
never fabricates bibliographic metadata. Supply vetted records as
`--metadata-json verified-references.json --output ...` to insert missing
entries automatically. `--renumber --output ...` is allowed only after the
audit is clean and only renumbers existing numeric citations and entries.

### Floating figures and Word fields

Normalize existing floating-image anchors without modifying the image bytes or
captions, then request Word to update field results when the output opens:

```bash
python scripts/normalize_floating_figures.py thesis.docx thesis-figures.docx --horizontal center --vertical top
python scripts/update_fields.py thesis-figures.docx thesis-final.docx
```

`update_fields.py` updates the document setting used by Word for TOC, REF, and
PAGEREF fields. Open the result in Word and accept its update prompt; render it
again afterwards because page-number and TOC changes affect layout.

### PDF formatting guides

When the school supplies a PDF guide in addition to a DOCX template, extract
page-cited evidence first:

```bash
python scripts/parse_pdf_rules.py school-guide.pdf --output pdf-requirements.json
```

Review the evidence, inspect diagram-only or scanned pages visually, and merge
only confirmed requirements into `school-rules.json`. A PDF extractor must not
infer layout rules from unverified text.

## Focused tools

- `scripts/format_heading.py`: apply only title and heading role rules.
- `scripts/format_tables.py`: apply table font, alignment, and cell alignment.
- `scripts/format_figures.py`: apply figure/table caption rules.
- `scripts/format_references.py`: format the reference heading and entries.
- `scripts/parse_rules.py`: inspect generated rules or regenerate them from a template.
- `scripts/complete_content.py`: find placeholders and apply approved content.
- `scripts/manage_references.py`: audit or safely renumber numeric citations.
- `scripts/normalize_floating_figures.py`: make anchored drawings non-overlapping.
- `scripts/update_fields.py`: mark TOC/cross-reference fields for updating on open.
- `scripts/parse_pdf_rules.py`: extract reviewable, page-cited rules from a PDF guide.

Read [references/docx-formatting.md](references/docx-formatting.md) before
formatting. For citation-list scope, also read
[references/citation-rules.md](references/citation-rules.md). For degree-level
structure, read [references/chinese-degree-thesis-workflow.md](references/chinese-degree-thesis-workflow.md).

## Safety and fidelity

- Do not alter the original template or source paper.
- Do not invent school requirements, citations, captions, reference entries, or
  content. Report anything the template does not reveal.
- Preserve text, field codes, section breaks, tracked changes, images, and table
  content. This skill changes layout and styles only.
- The JSON files in `templates/` are generic starting points only. Never present
  them as a school's submission standard when an official template is available.
