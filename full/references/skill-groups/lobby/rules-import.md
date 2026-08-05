# Rule import and packs

Stage the exact source; inspect, repair damaged pages, ingest, review candidates,
compile, test, install, and activate in order. OCR repair never invents numbers,
abilities, or unsupported rules. A missing standard implementation blocks until
fixed; it does not become homebrew.

For damaged text, call `rule_import(render_page)` and then checksum-bound
`rule_import(review_text)` before ingest. A text-only Agent may use corroborated
text or bounded context but must preserve digits, written quantities, and
heading depth. PDF and OCR cache remain immutable.

If all extractors omitted a physical page, an image-capable Agent may submit one
`rendered_page`-bound replacement with `old: ""` and the complete literal page
transcript as `new`. The server requires the page to be empty, exactly one
replacement, and a matching image checksum. A visionless Agent may later use
the reviewed text but cannot claim visual review.

Text visible in the rendered page is a source typo, not OCR damage: preserve it
and bind structured data to stronger same-page evidence. For a bad field in an
otherwise proven statblock slot, a text-only Agent may provide `ocr_corrections`
only from immutable staged text; an image-capable Agent may bind it to the image
checksum. Do not rewrite the remaining card.

Durable regressions store accepted repairs in `text_reviews` with normalized
text and evidence checksums. Replay the public transaction before ingest and
reject drift.

Profiles lock exact versions, dependencies, checksums, provider fingerprints,
and edition. Search, expand, and preserve exact citations and receipts.

For a reviewed portable `rule_pack`, skip PDF/OCR and candidate extraction. Use
`rule_import(import_package)`, inspect dependencies, and keep install separate
from Owner/DM activation. Export with `rule_pack_query(package)`; an inspected
`release_manifest` has no installation or activation authority.
