# Source content review

Use text reconstruction and bounded local OCR before requiring an image-capable
model. Keep the managed file checksum, exact page/chunk, printed heading, raw
text, normalized text, warnings, and corroboration evidence together.

For whole-page transcription damage, read the server-provided normalized,
native, and independent OCR variants. Submit only exact unique page replacements
through `rule_import(review_text)` or
`module_review(submit_transcript)`. A visionless Agent uses `cross_text` when two
text sources agree, or `agent_context` for a small spelling/case/heading repair
whose digits are unchanged. Only an Agent or human that actually inspected the
rendered page may use `rendered_page`. Revisions are checksum-bound audited views;
the staged source and OCR cache remain immutable.

Route statblocks by the campaign's locked edition. The bounded layout-OCR
`recover_statblock` facade currently recognizes only 2014 statblock grammar and
must fail closed for 2024. A complete 2024 indexed candidate uses
`content_kind="dnd5e_2024_statblock"` and the 2024 parser through text review;
an image-only 2024 card needs literal review by an image-capable Agent. Never
reinterpret one edition's headings, defenses, Challenge fields, or activities
with the other edition's parser.

When a custom card still needs semantic completion, let the Agent fill only the
facade's constrained fields from the returned evidence. Retry with a fresh
idempotency key; the server validates and stores the immutable review or
versioned solution.

An OCR failure does not require another creature-specific parser patch. For a
2014 statblock, first call `rule_import(render_page)` and compare the exact
page's normalized text, native text, independent OCR variants, and structural
card slots. A text-only Agent may retry
`rule_import(recover_statblock)` with the exact `page_number`,
`statblock_slot`, printed `name`, and `ocr_corrections`:

- `abilities` contains only complete `score (+/-modifier)` cells that occur in
  the staged page text;
- `text_replacements` contains an exact damaged span from that one OCR card and
  replacement text present in the staged page text.

The MCP verifies the replacement against the immutable source, requires the old
span to occur exactly once inside the selected card, reruns the parser and an
independent OCR corroboration path, and records the correction in the review
evidence. Reuse the same idempotency key only for an exact retry: the MCP must
return the stored full recovery response without rerunning OCR or asking the
Agent to review the same evidence again. An image-capable Agent may instead inspect the checksum-bound render
and submit a complete visual statblock review. If neither page text nor an
actually inspected image proves the missing fact, stop for external review.

Do not infer missing/conflicting numbers, silently acknowledge unresolved
warnings, or turn OCR output into a standard Core mechanic. An Agent without
image capability may use verified text recovery; genuinely unresolved visual
evidence remains external review.
