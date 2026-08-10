# Persistent NPC conversation

Use `npc_conversation` as the only Director-visible MCP facade and read
`../../host-integration-npc-conversation.md` before the first connected
multi-turn dialogue.

1. `open` with explicit participants and idempotency.
2. Rule `audience_facts` from current scene evidence before every `ingest`.
   Perception, comprehension, and response selection belong to the Agent; MCP
   never guesses them.
3. Send only returned `activation_ref` descriptors to
   `npc_conversation_worker(action="activate")`.
4. Treat worker output as a candidate. Rule publication audience (per segment
   when necessary), call `publish`, then show only MCP `publication`.
5. Resolve requested mechanics locally without suspending unrelated dialogue,
   then `ingest` a `resolution` event naming the completed
   `resolved_resolution_ids`.
6. Select actor-owned and mechanically derived listener candidates explicitly,
   `close` with the current conversation revision, and release workers.

Never expose the Host transport, private capsule, lease, raw proposal, intent,
truth posture, or basis refs. Never activate every witness. A perceived but
uncomprehended event must contain no raw speech in that actor's inbox.
