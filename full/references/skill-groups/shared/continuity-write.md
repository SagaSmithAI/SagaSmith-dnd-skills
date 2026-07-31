# Continuity writes

Use `memory_change(action="commit")` to atomically persist a completed narrative
unit: one event, objective fact revisions, actor-specific knowledge, and an
optional proportionate Snapshot. Preserve stable fact keys and expected fact
revision ids.

If a commit cites module evidence pinned by a `context_anchor`, include the
fresh `context_receipt` returned by the matching `continuity_context` read. A
stale, cross-branch, cross-principal, or incomplete receipt must be rejected and
re-read, not bypassed.

Write only durable consequences. Intent, possible future behavior, and DM
working notes remain context until fiction makes them true. Propagate knowledge
only to actors who actually learned it.
