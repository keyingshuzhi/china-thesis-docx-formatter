# Citation and Reference Rules

This skill formats the visual presentation of an existing reference list. It
does not infer missing sources or rewrite bibliographic data.

Before applying `format_references.py`, identify the standard from the school's
guide or the template (for example GB/T 7714, APA, or a journal rule). The
formatter can set paragraph font, hanging indent, alignment, and spacing. It
reports likely reference entries but cannot prove citation correctness.

`manage_references.py` recognizes bracketed numeric citations such as `[1]` and
`[1, 2]`. It can audit them against bibliography entries and, after a clean
audit, renumber existing labels. It must not invent a source from an orphaned
citation; require a user-supplied, authoritative bibliography record instead.
Provide that JSON mapping to `--metadata-json` when asking it to insert a
missing entry.
