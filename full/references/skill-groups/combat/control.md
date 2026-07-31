# Combat control and close

Join reinforcements only from reviewed canonical actors with source evidence,
entry timing, map placement, and current encounter revision. Joining is a
separate transaction and must not rewrite initiative history.

Close combat only after all pending choices, payments, concentration, death,
temporary effects, and outcome requirements are resolved. Use an audited
structured outcome; do not force a module ending from narration alone.

After `combat_end`, refresh the Play exposure, re-query character and campaign
state, then commit durable casualties, relationships, clues, loot, scene
progress, and manifest changes through normal Play continuity tools.

If a combat write returns `narrative_followup`, keep the mechanical result and
send each listed named NPC through the isolated portrayal workflow before its
next narrative decision. The follow-up never grants a free move/action or
implements a module-specific surrender, escape, or negotiation trigger.
