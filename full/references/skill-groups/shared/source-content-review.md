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

Do not infer missing/conflicting numbers, silently acknowledge unresolved
warnings, or turn OCR output into a standard Core mechanic. An Agent without
image capability may use verified text recovery; genuinely unresolved visual
evidence remains external review.
