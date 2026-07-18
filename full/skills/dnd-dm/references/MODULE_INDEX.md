# Module Index and Play

Full Runtime uses MCP-managed Markdown artifacts. Generate module text or provide a
configured source path, then call `module_import` in order: `stage`, `inspect`,
`validate`, `ingest`, `activate`. Do not let the agent read arbitrary local module
paths in Full Runtime.

Every stage uses the same D&D parser profile. Pass a stable stage-specific
`idempotency_key` and reuse it only for the exact same retry.

Use `module_query(view="index")` to choose scenes. For each player action call
`module_query(view="current")` with `scope_id`, then
`module_query(view="scene")`. Search is only a
selector: `module_search` must be followed by `module_expand` before a result is
used as module truth.

Scopes are `party`, `group:<id>`, and `player:<id>`. A player or group inherits the
party current scene until its own `module_set_progress` call establishes a separate
progress record. Update progress with the complete merged `state`, `current_room`,
status, and percent; do not reveal rooms or facts outside that scope.

When leaving a scene, write it as `completed` with progress `100`, then set the
next scene to `current`. Record a snapshot before chapter transitions.
