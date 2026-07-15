# Module Index and Play

Full Runtime uses MCP-managed Markdown artifacts. Generate or provide module text,
then call `module_write`, `module_inspect`, and `module_import`. Do not let the
agent read arbitrary local module paths in Full Runtime.

`module_inspect` and `module_import` use the same D&D parser profile. Pass a
stable campaign-wide `idempotency_key` to the import and reuse it only for the
exact same retry.

Use `module_index` to choose scenes. For each player action call
`module_current(campaign_id, scope_id)`, then `module_read_scene`. Search is only a
selector: `module_search` must be followed by `module_expand` before a result is
used as module truth.

Scopes are `party`, `group:<id>`, and `player:<id>`. A player or group inherits the
party current scene until its own `module_set_progress` call establishes a separate
progress record. Update progress with the complete merged `state`, `current_room`,
status, and percent; do not reveal rooms or facts outside that scope.

When leaving a scene, write it as `completed` with progress `100`, then set the
next scene to `current`. Record a snapshot before chapter transitions.
