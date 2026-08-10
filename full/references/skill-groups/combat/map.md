# Combat positioning

`combat_start` fixes `positioning_mode="grid"|"agent"` for the whole
encounter. Never switch modes mid-combat.

In grid mode, provide a compiled map and a coordinate for every participant.
Patch it only through `combat_map_patch` from reviewed scene/map evidence or an
explicit bounded DM spatial ruling. The engine owns movement distance, range,
line/area geometry, obstruction, cover, adjacency, threats, and friendly fire.
Missing coordinates are invalid grid input.

In agent mode, provide neither a battle map nor coordinates. The Agent infers
whether movement and targets are legal, what is in range or blocked, who
threatens whom, and whether an area includes allies. Supply those decisions as
the exact action-specific `spatial_facts`; the engine still owns rolls, action
economy, damage, resources, effects, and commits.

Keep public and DM-only layers separate. Walls, blocking, difficult terrain,
occupancy, elevation, cover, hazards, and actor placement must retain source or
ruling provenance. A decorative image is not mechanical geometry.

In grid mode, map revision participates in movement and reaction validation.
Re-query the map after any patch, join, restore, or movement conflict.

Rendering is a projection, not another map model. Request it through
`combat_query(view="render")`. A grid result may depict the current projected
geometry and package-owned actor portraits. An Agent-mode result must remain a
nonspatial initiative card with no invented coordinates. Send `party_public`
images to shared channels; do not send a private `caller` render there.
