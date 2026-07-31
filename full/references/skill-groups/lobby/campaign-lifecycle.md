# Campaign lifecycle

On the campaign-bound exposure, grant membership through `access_grant` and
keep the current branch explicit. Campaign administration is Owner/DM work;
never infer authority from a display name or model-authored principal.

Use `campaign_change` for campaign state, clock, advancement, party rest,
currency, loot, and world effects. Use `playthrough_manifest` for route and
ending audit state. Use branch and Snapshot facades for isolated alternatives;
restore only verified snapshots and immediately discard pre-restore context.

Campaign rules are locked by exact pack versions and fingerprints. Core relock
is an explicit checkpointed runtime upgrade, never an automatic recovery.
