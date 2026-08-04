# Module import

Stage all campaign documents and assets under one module identity. Inspect OCR,
review uncertainty, ingest, validate the runtime manifest, then activate.

Extract party range, levels, advancement, pregenerated-PC conditions, endings,
encounters, scenes, actors, items, maps, and exact references. Unresolved party
size or play-critical content enters DM review.

Prose is not executable. Preserve narrative evidence for Agent adjudication;
each mechanical card needs a native mechanic, reviewed plan, or exact-source
Agent-ruling contract before activation/export.

For damaged text, call `module_review(render_transcript)` then
`module_review(submit_transcript)` before ingest. Text-only Agents need
cross-text evidence or bounded context with unchanged digits; only a visual
reviewer cites the image. Refresh revisions; never mutate the PDF.

Bind cast and pregenerated PCs to stable Atlas keys. Export/import packages via
public tools, never database rows; packages exclude campaign progress, memories,
branches, and Snapshots.
