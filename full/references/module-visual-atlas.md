# Module Visual Atlas Review

Use this workflow when imported module text identifies locations but a map,
diagram, or handout carries topology that the text parser cannot safely infer.
Run it in `lobby` while preparing a module, or in `play` before relying on a
missing edge. Load `lobby.modules` or `play.scene` first.

## Review sequence

1. Call `module_query(view="index")` and inspect `spatial.locations` and
   `spatial.connections`. An empty connection list means unknown topology.
2. Call `module_query(view="assets", payload={"module_id": ...})`. Select the
   imported `application/pdf` asset; do not read an arbitrary local path.
3. Locate a candidate page from scene page ranges or expanded source text, then
   call `module_page_render(campaign_id, module_id, page_number,
   source_asset_id)`. Inspect the returned image itself. Text extraction, room
   numbering, heading order, and a generic cross-reference are not visual proof.
4. Re-read `module_query(view="current" | "progress")` and use the current
   scene/scope `state_version` (`0` only when no row exists).
5. Call `module_set_progress` with a fresh idempotency key and
   `spatial_review`. Omit `status` and `progress` when they should remain
   unchanged. Use `mode="merge"` to add/correct the submitted edges or
   `mode="replace"` only after reviewing the complete replacement set.
6. Re-read the scene or current progress. Use an edge only when it returns with
   `confidence="reviewed_image"` and evidence containing the asset checksum,
   source page, reviewer, and active branch id. Create a snapshot after a
   material atlas review.

```json
{
  "campaign_id": "campaign-id",
  "scene_id": "current-or-spatial-scene-id",
  "scope_id": "party",
  "expected_state_version": 3,
  "idempotency_key": "review-dungeon-map-page-22-v1",
  "spatial_review": {
    "schema_version": 1,
    "mode": "merge",
    "source_asset_id": "imported-pdf-asset-id",
    "page_number": 22,
    "note": "Reviewed the printed dungeon plan.",
    "connections": [
      {
        "from": "d5-welcome-to-the-dungeon",
        "to": "d6-bloated-corpse",
        "bidirectional": true,
        "kind": "passage",
        "observation": "The map visibly draws an open corridor between D5 and D6."
      }
    ]
  }
}
```

Allowed connection kinds are `passage`, `door`, `secret_door`, `stairs`,
`portal`, and `other`. Each endpoint must name exactly one location in the same
module. The observation must describe only what is visible on that page; do not
smuggle inferred secrets, encounter outcomes, or geometry into it.

## Persistence and visibility

Reviewed topology is stored in scoped scene progress, so snapshot restore and
branch checkout restore the corresponding atlas review. It is not written back
into immutable imported module metadata and does not leak into sibling branches.
Every Agent/session must open its own exposure, but all authorized Agents read
the same branch state through MCP.

The rendered page is DM/owner evidence. Do not show it or keeper-only topology to
players unless the module and campaign state establish that the party possesses
that map or handout. A reviewed scene edge helps scene traversal and combat-map
provenance; it does not establish walls, cover, line of sight, difficult terrain,
blocked cells, or exact token positions. Supply those facts separately when the
source or DM establishes them.
