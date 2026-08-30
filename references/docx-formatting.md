# DOCX Formatting Notes

`python-docx` exposes most paragraph, run, table, section, header, and footer
properties. Some Word properties—including East Asian font names—live in the
underlying OOXML; the scripts set these explicitly where needed.

Generated rules distinguish between observed template values and assumptions:

- `source`: the template filename and extractor version;
- `page`: first section page geometry, plus a warning when sections differ;
- `role_rules`: styles mapped to semantic roles;
- `table_rules`, `figure_rules`, and `reference_rules`: observed or conservative
  rules that can be reviewed before use;
- `extraction_warnings`: details that need manual review.

Use named Word styles whenever the template supplies them. For documents that
use direct formatting, the extractor samples the most common direct appearance
and marks the result for review. Never rely on a rendered appearance alone when
the underlying styles conflict.

Floating images are OOXML `wp:anchor` elements. The normalizer only adjusts
anchor positioning and overlap flags; it does not rasterize, replace, crop, or
move captions. TOC and cross-reference fields are dynamic Word fields: setting
`w:updateFields` asks Word to refresh them on open, but the resulting page
layout must still be rendered and inspected.
