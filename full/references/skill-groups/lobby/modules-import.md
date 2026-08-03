# Module import

Stage the PDF, Markdown, maps, character sheets, guides, and auxiliary assets
under one module identity. Inspect normalization and selective OCR results,
review uncertain content, ingest the source index, validate the runtime
manifest, then activate it.

Extract recommended player range, start/end levels, advancement method,
pregenerated-character conditions, ending conditions, encounters, scenes,
NPCs, monsters, items, maps, and source references. Unresolved recommended
party size or content critical to play enters explicit DM review.

Module prose is not automatically executable. Preserve exact narrative
evidence for Agent adjudication. Before activation or export, every mechanical
card must already carry a native mechanic, a reviewed typed plan, or an
exact-source direct Agent-ruling contract; never defer authoring to live use.

Bind imported cast, encounter actors, and pregenerated PCs to stable Scene Atlas
keys before export. Use `module_query(view="package")` for a self-contained
portable module and `module_import(action="import_package")` for re-ingestion;
never copy database rows. The package carries source/index/assets/reviews/cards,
not campaign progress, memories, branches, or Snapshot state.
