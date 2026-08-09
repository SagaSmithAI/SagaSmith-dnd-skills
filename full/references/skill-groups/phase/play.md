# Phase: Play

Play is live non-combat scene resolution. At each turn:

1. Query the current scene, branch, campaign revision, relevant continuity, and
   actor knowledge for the intended audience.
   Cross a changed `host_context_binding` before continuing. Use the matching
   bounded purpose for autonomous actors, factions, source interpretation,
   rulings, or player-facing rendering; never reuse a DM bundle for a player.
2. Retrieve exact module/rule evidence before factual narration or settlement.
3. Resolve automatic standard mechanics with the engine; use Agent DM reasoning
   only at the declared ruling boundary.
4. Apply state changes through the exposed facade.
5. At meaningful scene completion, atomically record event, facts, actor
   knowledge, manifest progress, and a proportionate checkpoint.

For connected NPC dialogue, open one MCP conversation, retain one isolated host
worker per NPC, publish only MCP-validated publications, and atomically close
the transcript before any authoritative mechanic or scene mutation.

Start Combat only from reviewed canonical actors and scene evidence. Do not
carry a pre-restore context bundle into resumed narration.
