# Persistent NPC conversation

Use this during Play for connected dialogue. It requires a host with genuine,
isolated subagents; there is no shared-context or MCP-hosted-model fallback.

1. Check `server_capabilities.npc_conversations`, then call `conversation_open`
   once with all present actors. MCP creates one private logical runtime per NPC
   from current campaign, branch, scene, character, ActorKnowledge,
   relationship, goal, and commitment authority.
2. Append each player speech/action with `conversation_ingest`. MCP derives who
   perceived and understood it and returns public activation descriptors. Do
   not let the Director retrieve private actor capsules.
3. Dispatch chosen activations to an actor-isolated worker. SagaSmith Agent uses
   `npc_conversation_worker(action="activate")`, which privately calls
   `npc_activation_checkout`, reuses that NPC's message context, submits its v2
   proposal, and returns only MCP `publication`.
4. Publish only `publication`. Never expose the proposal, private intent,
   decision summary, truth posture, basis refs, working deltas, bootstrap,
   handles, or leases.
5. Player lines and publications immediately enter the draft journal and later
   NPC inboxes; they do not write campaign authority per turn. Select explicit
   per-actor delta indexes and call `conversation_close` once at the natural
   boundary, then release every worker.
6. Any mechanic, event write, branch/Snapshot change, actor revision, scene
   mutation, or `SESSION_STALE` ends the session. Commit completed dialogue when
   still fresh, run ordinary public tools, and open a new conversation. Version
   1 never hot-refreshes authority inside a conversation.

Every speakable byte must be in proposal v2 `utterance_segments`, with speech
act, truth posture, actor-owned basis refs, targets, language, and delivery.
Factual/deceptive assertions, revelations, and lies require basis refs;
mechanical actions require resolution requests. MCP derives the safe
publication and atomically commits the exact public transcript plus accepted
ActorKnowledge, relationships, goals, and commitments.

Keep model prefixes ordered as stable contract, scene context, private NPC
bootstrap, that NPC's perceived journal, then current activation. Never share
post-bootstrap KV between NPCs. KV loss only requires rebuilding from MCP
semantic state. Dispose caches on close/abort or any authority/model/schema
change. See `references/host-integration-npc-conversation.md` for the complete
host contract.
