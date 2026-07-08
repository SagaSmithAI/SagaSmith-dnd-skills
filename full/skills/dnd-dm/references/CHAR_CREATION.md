# Character Creation And Advancement

## Creation Flow

1. Read the campaign rule profile first: edition, locale, publications, and options.
2. Confirm the player's concept, level, ability-score method, class, species, background, proficiencies, languages, tools, equipment, and spells.
3. Roll only through the CLI when random results are required.
4. Build a draft sheet with abilities, level, AC, HP, class/species/background, proficiencies, and spells. Do not persist until the player confirms.
5. After creation, read the saved character and verify derived values, resources, and campaign/player bindings.

```powershell
sagasmith-dnd character create --campaign <id> --name "<name>" --type pc --player "<player>" --sheet '<json>' --json
sagasmith-dnd character list --campaign <id> --json
sagasmith-dnd character show --id <character-id> --json
```

## Inventory And Runtime Items

Do not put authoritative equipment or backpack state in `sheet.inventory`.

- Use `item ... --json` for the campaign ledger: backpack ownership, treasure, currency, containers, attunement, identification, and consumables.
- Use Actor-owned `game-item` and `game-activity` documents for weapons, spells, class features, and combat-capable equipment.
- The CLI does not import `sheet.inventory` and does not return an inventory-managed flag.
- Runtime command IDs stay in English even during Chinese narration; localized labels may follow fvtt-cn terminology.

```powershell
sagasmith-dnd item add --campaign <id> --name "Potion of Healing" --owner-type character --owner-id <character-id> --quantity 2 --json
sagasmith-dnd game-item create --campaign <id> --actor <actor-id> --name "Longsword" --type weapon --payload '{"equipped":true}' --json
sagasmith-dnd game-activity create --item <item-id> --name "Slash" --type attack --payload '{"activation":{"type":"action"},"system":{"attack_bonus":5,"damage":"1d8+3","damage_type":"slashing"}}' --json
```

## Advancement

Advance from the existing actor/character state. Do not rebuild and overwrite prior choices.

- Use structured advancement steps for level, hit points, class features, spell slots, proficiencies, and granted items.
- After advancement, call `actor prepare` and verify derived AC, HP, proficiency, saves, skills, resources, and activities.
- Record the advancement event and create a snapshot.

```powershell
sagasmith-dnd advancement apply --campaign <id> --actor <actor-id> --payload '<json>' --json
sagasmith-dnd actor prepare --campaign <id> --actor <actor-id> --json
sagasmith-dnd save create --campaign <id> --label "Level up" --json
```
