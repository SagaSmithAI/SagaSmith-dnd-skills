# Image-only Module Content Review

Use this workflow when an imported PDF visibly contains a creature statblock but
the PDF text layer and imported scene omit the card. This is a source-recovery
path, not permission to invent mechanics. Perform actor preparation in `lobby`
and never create or repair a required actor after combat has begun.

## Ordered workflow

1. Use `module_query(view="index" | "scene")` to locate the appendix scene and
   confirm that the normalized text does not contain an executable statblock.
2. Use `module_query(view="assets")`, select the managed PDF, and call
   `module_page_render` for the cited page. Inspect the returned image itself.
3. Transcribe only visible card facts into canonical English 2014 statblock
   Markdown. Preserve the exact name, size/type/alignment, AC, HP formula, speed,
   six abilities, listed saves/skills/defenses/senses/languages, CR/XP, headings,
   attack bonus, reach/range, damage dice, damage bonus, and damage type. Do not
   fill an absent field from memory or a similar creature.
4. Call `module_content_review` with the appendix `scene_id`, stable
   `content_key`, normalized Markdown, managed PDF or rendered-image asset,
   1-based page, a literal visual observation, and a fresh idempotency key.
5. Stop if validation rejects the card. If it returns `mixed`, review every
   warning and keep unresolved mechanics visible. `automatic` means only that
   the transcribed mechanics represented by the current engine are executable.
6. Re-read the immutable record with `module_query(view="content",
   payload={"review_id": ...})`; verify its asset checksum, page, scene, reviewer,
   content checksum, and `confidence="reviewed_image"`.
7. Create each required campaign actor with
   `character_create_from(mode="module_statblock", payload={"campaign_id": ...,
   "review_id": ..., "name": ..., "character_type": "monster"})`. Re-read the
   actor and verify AC, HP, attacks, source refs, and unresolved rules before
   adding its canonical id to the scene-readiness manifest.

If a room names a standard rule statblock and then states a small instance change,
import the exact standard source and use `character_create_from(mode="statblock")`
with `payload.variant`. Do not transcribe a second whole card or patch the resulting
sheet. Every variant requires a module `source_ref`; the runtime accepts only
explicit current/maximum HP, AC, languages, action removal, and narrow weapon-action
overrides. For example, a wounded unarmored Noble can set `current_hit_points`,
`armor_class`, `languages`, and `remove_actions`, while an animated gauntlet based
on Flying Sword can rename its weapon and change only the cited damage type.

```json
{
  "mode": "statblock",
  "payload": {
    "campaign_id": "campaign-id",
    "source_id": "exact-noble-rule-source-id",
    "name": "Klim Jhasso",
    "character_type": "npc",
    "variant": {
      "source_ref": "module-chunk:d12-banes-altar",
      "current_hit_points": 1,
      "armor_class": 10,
      "languages": ["Common", "Elvish"],
      "remove_actions": ["rapier"]
    }
  },
  "idempotency_key": "create-klim-jhasso-d12-v1"
}
```

Reject a variant if the cited source does not explicitly establish every change,
if an action id is ambiguous, or if the desired change is outside the whitelist.
The base source and variant source must both remain visible in actor provenance.

```json
{
  "campaign_id": "campaign-id",
  "module_id": "module-id",
  "scene_id": "appendix-creature-scene-id",
  "content_key": "necromite-of-myrkul",
  "content_kind": "dnd5e_2014_statblock",
  "normalized_content": "# Necromite of Myrkul\n\n*Medium humanoid (human), neutral evil*\n...",
  "source_asset_id": "managed-pdf-or-rendered-page-asset-id",
  "page_number": 181,
  "observation": "The complete Necromite of Myrkul card is visible in the upper-left column.",
  "idempotency_key": "review-necromite-page-181-v1"
}
```

Numeric melee, ranged, weapon, and spell attacks with explicit to-hit, range or
reach, dice, bonus, and type can settle automatically. Narrative traits,
ambiguous multiattacks, incomplete spellcasting, recharge/choice semantics, or
other unsupported effects remain warnings or DM rulings. Never erase a warning
to make readiness pass.

The review belongs to the imported campaign module and is immutable provenance;
it is not branch-scoped narrative state. Actors created from it retain the review
id, module/scene ids, page, and asset checksum in their notes. A corrected visual
transcription creates a new review record and new actor preparation; do not mutate
the old evidence or silently rewrite an active combatant.
