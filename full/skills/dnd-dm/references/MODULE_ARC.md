# Module Arc

Static module text remains external to saves. At a chapter transition, call
`campaign_event(action="add")` with `event_type: "chapter"`, persist scoped scene progress with
`module_set_progress`, then call `snapshot_create`.

Restoring a snapshot forks a new branch; it never rewrites the static module source
or deletes future campaign history. Keep scene progress, clues, faction state, and
player-caused world changes in MCP campaign state rather than modifying module text.
