# Host integration: persistent NPC conversations

This is the primary Play-mode contract for consecutive dialogue. It requires a
host with real subagents. The MCP owns semantic actor runtimes and authority;
the host owns model workers and provider KV caches. There is no server-managed
inference and no single-model fallback.

## Required host capabilities

Before opening a conversation, verify
`server_capabilities.npc_conversations.execution_mode` is
`"client_subagents_required"`. The host must provide:

- one isolated message context per `conversation_id + actor_runtime_id`;
- zero tools, workspace, parent history, skills, and durable host memory inside
  each NPC worker;
- host-level routing so the Director model never receives an actor's private
  checkout capsule or raw proposal;
- structured JSON output and actor-scoped dispatch;
- persistent worker contexts until close/abort, followed by explicit disposal.

If those guarantees are unavailable, do not start the conversation protocol.
Do not silently let one model context portray every NPC.

## Director flow

1. Open `play.npc_conversation` and call
   `conversation_open(campaign_id, participant_actor_ids, scope_id, query)`.
   Participants are explicit, same-campaign actors. MCP creates one durable
   logical runtime and one initial private authority snapshot for every NPC.
2. Send each player-observable utterance/action through
   `conversation_ingest`. The MCP derives perception, language understanding,
   direct targets, and NPC activations. The Director sees only public activation
   descriptors and opaque `worker_handle` values.
3. Dispatch each chosen activation to the host NPC worker bridge. In
   SagaSmith Agent use `npc_conversation_worker(action="activate", ...)`. The
   bridge performs checkout and submit inside host code; never copy the private
   capsule into the Director prompt.
4. Publish only `npc_activation_submit.publication`. Never publish the worker's
   proposal, `private_intent`, truth posture, decision summary, working deltas,
   basis refs, or bootstrap.
5. If the result is `resolution_required`, stop dialogue. Close the completed
   transcript, execute the ordinary public mechanic/state tools, and open a new
   conversation from fresh authority. Version 1 does not hot-refresh a running
   conversation across an authoritative mutation.
6. At the natural boundary, choose explicit working-delta indexes per actor and
   call `conversation_close` once. The MCP commits the exact public transcript,
   a server-derived retrieval summary, and accepted ActorKnowledge/actor-state
   changes atomically. Then call the host worker's `release` action.

Every active call checks the campaign revision, branch, latest event sequence,
and participant actor revisions. `SESSION_STALE` means the draft must not
continue. Close if the completed transcript can still be committed under the
same authority; otherwise abort and reopen after handling the external change.

## Actor worker and cache layout

The first checkout returns `bootstrap`; later checkouts use the worker's inbox
cursor and normally return only working state plus new inbox events. Keep the
model prefix in this order:

1. stable zero-tool NPC contract and proposal v2 shape;
2. conversation-scene public context;
3. this NPC's private initial actor/knowledge/relationship/goal/commitment data;
4. this NPC's actually perceived conversation events;
5. the current activation.

This produces a cache tree with a shared contract/scene prefix and one private
branch per NPC. Once private actor branches diverge, never share their suffix KV
blocks. Provider cache loss is a performance event, not a continuity event:
recreate the worker from bootstrap plus the MCP actor journal. Track
`cached_tokens`, but never store provider KV as campaign authority.

Invalidate and dispose the worker on close/abort, branch or Snapshot changes,
scene/actor/knowledge revisions, principal changes, model/prompt/schema changes,
or any authoritative mechanic. Opaque handles, leases, signatures, and receipts
belong in host runtime state, not in the model's stable prompt prefix.

## Proposal v2 boundary

`npc-conversation-proposal.v2` has no free `utterance.text`. Every speakable
byte is an `utterance_segments[]` item carrying speech act, truth posture,
basis refs, targets, language, and delivery. Factual/deceptive
assert/reveal/lie segments require an allowed actor basis. Mechanical actions
require `resolution_requests`. The MCP validates the exact activation lease,
actor runtime, basis refs, targets, working deltas, and authority before deriving
publication.

Player lines and validated publications enter the draft journal immediately so
later NPC turns perceive them. They do not mutate campaign authority until
`conversation_close`. A listener receives a claim with provenance (for example,
"Mara said X"), not automatic proof that X is objectively true.
