# Core: bootstrap

Use SagaSmith only through the public MCP contract. On a cold connection:

1. Read `sagasmith://bootstrap` when MCP resources are available.
2. Call `storage_status`, `server_capabilities`, and
   `campaign_query(view="list")`.
3. For an existing campaign call `campaign_query(view="resume")`.
4. Open exactly one exposure for the trusted principal. Before campaign
   creation, load only `lobby.bootstrap`; after creation reopen with the new
   `campaign_id`.
5. Search, inspect, then load capability groups. Do not retain an exposure id
   after opening its replacement.
6. Refresh `tools/list` after `tools/list_changed`; otherwise dispatch only
   loaded domain tools through `exposure_call`.

Use `skill_query(action="plan")` after opening or changing an exposure and read
every `required_now` document before performing unfamiliar work.
