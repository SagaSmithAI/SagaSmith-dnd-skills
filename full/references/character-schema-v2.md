# D&D Character Schema v2

Runtime mode stores every PC, NPC, and monster as a `Character` record with the
same validated documents. `character_type` is `pc`, `npc`, or `monster`; it does
not change the required sheet shape.

## Authority

- `sheet` is the authoritative mechanical state.
- `notes` is authoritative narrative state: description, important memories,
  relationships, and goals.
- `derived` is returned by `character show`; it is calculated from the sheet and
  must never be edited directly.
- `campaign.state.party.inventory` is the authoritative shared wallet and stash.
- Every new sheet and notes document has `schema_version: 2`.

## Sheet

`sheet` contains these required top-level blocks:

```json
{
  "schema_version": 2,
  "edition": "2014",
  "progression": {},
  "abilities": {},
  "skills": {},
  "combat": {},
  "traits": {},
  "resources": {},
  "spellcasting": {},
  "content": {},
  "conditions": [],
  "effects": [],
  "inventory": {}
}
```

`progression` records level, XP, classes, subclass, hit die, background, and
species. `combat` records current/max/temp HP, AC, initiative, all movement modes,
hit dice, death saves, and exhaustion. `traits` records languages, proficiencies,
damage defenses, condition immunities, and senses. `resources` is a named pool
with `value`, `max`, `recovers_on`, and `source_key`.

## Spells And Limited Features

`content.spells` records a spell's source, level, grant, and access flags:
`known`, `prepared`, `always_prepared`, `in_spellbook`, `ritual_available`, and
`at_will`. The authoritative daily choice is
`spellcasting.preparation.selected_spell_ids`.

- Clerics, druids, paladins, and prepared casters use `mode: "prepared"`.
- Wizards use spellbook membership separately from daily preparation.
- Known casters use `mode: "known"`; do not fabricate a prepared list.
- Always-prepared spells are returned as prepared without consuming a selection.
- `content.features`, `content.feats`, and `content.activities` use the same
  source/description/uses/choices shape for limited class, racial, item, and
  feat capabilities.

## Inventory And Wallet

`inventory.wallet` is `{ "cp": 0, "sp": 0, "ep": 0, "gp": 0, "pp": 0 }`.
Balances are non-negative integers. Gems, trade bars, keys, and unusual currency
are items, not wallet balances.

Each item has a stable `id`, `name`, `kind`, `quantity`, `weight_oz`, `price_cp`,
short `description`, `source_key`, container/equipment state, identification,
attunement, condition, uses, charges, and type-specific `mechanics`.

Equipment slots are `armor`, `shield`, `main_hand`, `off_hand`, `head`, `neck`,
`cloak`, `gloves`, `boots`, `ring_1`, and `ring_2`. The slot map and each item's
`equipped` / `equipped_slot` fields must agree. Use `character equipment equip`
or `unequip`; never set those fields through an inventory patch.

Armor and shields have strict mechanics:

```json
{
  "armor": {
    "base_ac": 14,
    "dexterity_mode": "max",
    "dexterity_max": 2,
    "magic_bonus": 0
  },
  "shield": { "ac_bonus": 2, "magic_bonus": 0 },
  "magic_item": { "ac_bonus": 1 }
}
```

Armor uses `dexterity_mode: "none"`, `"full"`, or `"max"`; `dexterity_max` is
required only for `"max"`. Armor may only occupy `armor`, shields only `shield`,
and rings must be `magic_item` records in a ring slot.

`derived.armor_class` is calculated in this order: explicit `combat.ac.override`;
otherwise armor or `combat.ac.base`; then shield, equipped magic-item AC bonuses,
and supported active effects. `derived.armor_class_breakdown` explains every
applied source. A supported effect change uses
`{ "path": "derived.armor_class", "mode": "add|override", "value": <integer> }`.
Other effect changes remain in `derived.unresolved_rules` for DM adjudication.

Containers are ordinary `kind: "container"` items. Items reference their parent
with `container_id`; containers cannot form cycles. Shared party goods use the
same inventory structure.

## Effects And Narrative

Effects record source, active state, a declared duration period/remaining count,
structured changes, and a short description. The runtime lists effect changes it
cannot derive automatically in `derived.unresolved_rules`; the DM must read the
rules before narrating their result.

`notes.profile.summary` is the one-paragraph public description. NPCs require it;
monsters should also receive a concise appearance/behavior description. Important
dialogue facts use `notes.memories`, with `kind`, `summary`, `importance` 1-5,
participants, optional source event, visibility, and status. Do not store every
line of dialogue as a memory.

## Mutation Commands

```powershell
sagasmith-dnd character show --id <id> --json
sagasmith-dnd character inventory add --id <id> --payload '<item-json>' --json
sagasmith-dnd character inventory transfer --id <source> --target <target> --item <item-id> --amount <n> --json
sagasmith-dnd character wallet credit --id <id> --denomination gp --amount 10 --json
sagasmith-dnd character wallet transfer --id <source> --target <target> --denomination gp --amount 5 --json
sagasmith-dnd character equipment equip --id <id> --item <item-id> --slot-name main_hand --json
sagasmith-dnd character spell prepare --id <id> --spell <spell-id> --json
sagasmith-dnd character effect add --id <id> --payload '<effect-json>' --json
sagasmith-dnd character memory add --id <id> --payload '<memory-json>' --json
sagasmith-dnd party inventory deposit --campaign <id> --id <character-id> --item <item-id> --json
sagasmith-dnd party wallet deposit --campaign <id> --id <character-id> --denomination gp --amount 5 --json
```

Use `character update --sheet/--notes` only for a reviewed complete draft or a
deliberate full-sheet change. Never hand-edit one inventory entry, wallet balance,
prepared spell, effect, or memory through a raw sheet replacement during play.
