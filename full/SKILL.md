---
name: sagasmith-dnd-suite
description: "Cross-platform D&D 5e 2014/2024 game-master and campaign workflow using the SagaSmith JSON CLI."
version: 2.0.0
---

# SagaSmith D&D Suite

This repository is an Agent Skill, not a Python runtime. It operates
`sagasmith-dnd` through shell commands that always include `--json`.

## Startup

1. Run `sagasmith-dnd doctor --json`.
2. If it succeeds, use Runtime mode and load the relevant child Skill.
3. If the command is missing, use Portable mode: static SRD lookup and guidance only.
4. Never claim that Portable mode persisted a campaign, character, save, or memory.

## Included Skills

- `skills/dnd-dm`: play, adjudication, rule/module retrieval, and narration.
- `skills/dnd-campaign-manager`: campaign, character, save, and memory lifecycle.

Module generation is maintained separately in `SagaSmith-module-gen-skills`.

## Invariants

- Keep the active `campaign_id`, edition, and locale explicit.
- Never mix 2014 and 2024 rules unless the user explicitly requests comparison.
- Search first, then expand only the selected rule or module chunk.
- Treat CLI stdout as one JSON envelope; logs belong to stderr.
- On `ok:false`, branch on `error.code`; do not infer success.
- Runtime character state uses `sheet v2` / `notes v2`; load
  `references/character-schema-v2.md` before creating or mutating a character.
- Do not load entire rulebooks or modules into context.

See `references/cli-contract.md` and `references/workflows.md`.
