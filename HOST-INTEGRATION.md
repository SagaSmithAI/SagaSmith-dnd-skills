# Host integration

SagaSmith Full Runtime has two independent requirements:

1. install or discover this skill repository; and
2. connect the `sagasmith_dnd` MCP server.

The MCP initialization instructions and `sagasmith://bootstrap` resource are
the authoritative zero-knowledge entry point. A host that does not expose MCP
resources can call the always-visible `skill_query` tool with
`action="plan"`, then read every `required_now` document. `outline`,
`section`, and `search` provide additional task-specific depth.

## Identity boundary

Never expose `principal_id` as a model-controlled authorization choice.

- A trusted multi-user host must remove that field from model input and inject
  the authenticated platform identity on every call.
- A single-user stdio process should set
  `SAGASMITH_DND_MCP_BOUND_PRINCIPAL_ID=<stable-user-id>`. The server then
  overwrites every model-authored principal value.
- `system:local` is suitable only for an explicitly trusted local process.

## Codex

The repository includes `.codex-plugin/plugin.json`,
`skills/sagasmith-dnd-suite/SKILL.md`, and a project-discovery wrapper under
`.agents/skills/`. Configure the MCP server separately with the executable
`sagasmith-dnd-mcp`; set `SAGASMITH_DND_SKILLS_DIR` to this repository root
and bind a stable principal.

## Claude Code

The repository includes `.claude-plugin/plugin.json`, `skills/`, and
`.mcp.json`. `${CLAUDE_PLUGIN_ROOT}` points the MCP server back to the bundled
full skill corpus. The `sagasmith-dnd-mcp` console command must already be
installed in the environment Claude Code launches.

## OpenClaw

Install the repository root as a skill; root `SKILL.md` is intentionally
self-contained in its first screen and uses `{baseDir}` for canonical paths.
Configure the MCP server in OpenClaw's MCP settings and inject the authenticated
principal in the adapter. Do not pass a chat-authored principal through.

## Hermes

The repository root works as a direct skill source, while
`skills/sagasmith-dnd-suite/` supports a tap-style repository layout. Configure
the native MCP connection separately. Hermes can process
`tools/list_changed`; if a particular integration does not refresh, the
bootstrap explicitly directs it to `exposure_call`.

## Bounded context isolation

On `campaign_query(view="resume")`, store the returned
`host_context_binding`. If campaign, authenticated principal, role, audience,
branch, restore state, or its derived `context_epoch` changes, stop later calls
from that model response and rebuild without old messages, summaries,
workspace/Dream memory, retrieval, receipts, or tool results.

Hosts should provide one synchronous, fresh, zero-tool model call for each
signed bounded semantic bundle: actor turn, audience render, faction turn,
source interpretation, or bounded DM ruling. SagaSmith Agent provides
`isolated_evaluate` and retains `portray_npc` for rich NPC dialogue. Other hosts
must implement the native or logical contract in
`full/references/host-integration-bounded-context.md`; a generic background
subagent is unsafe because it may inherit tools, workspace, or history and
return after the signed receipt becomes stale.

The evaluator returns a proposal only. The parent validates it through
`bounded_evaluation(action="validate")`, resolves mechanics through public MCP
tools, rereads changed context, selects accepted deltas, and commits. For player
audiences, publish only the validator's exact `publication.text`. Hosts without
image capability are supported because reviewed OCR/module evidence is text
before it enters the bundle. See `full/references/host-integration-npc-turn.md`
for the additional NPC-dialogue commit contract.

## Required cold-start smoke test

For every host:

1. initialize MCP and verify `capabilities.tools.listChanged=true`;
2. verify the first `tools/list` contains only the core discovery tools;
3. verify
   `server_capabilities.zero_knowledge_bootstrap.phase_skill_plan.available=true`,
   then read
   `sagasmith://bootstrap` or call `skill_query(action="plan")` and read every
   `required_now` document; an unavailable plan is an installation failure;
4. call `campaign_query(view="resume")` for an existing campaign and verify
   its phase/role plan;
5. open a campaign exposure, inspect one facade action contract, load one
   group, read the returned `skill_plan_delta`, and verify either native
   refresh or `exposure_call`;
6. switch Play → Combat → Play and verify phase-only groups change while
   checksum-satisfied Core groups are not reread;
7. verify a forged `principal_id` cannot change the authenticated identity.
8. verify the first campaign binding causes a host context reset and the same
   binding does not loop; then change branch or audience and verify another reset;
9. request one bounded bundle, verify the evaluator has zero exposed tools and
   no child-session persistence, validate its proposal through MCP, then reject
   the same receipt after an event;
10. request one NPC bundle and verify its actor knowledge cannot be supplemented
   by public world truth or parent history.

After updating the installed Skills files, call
`skill_query(action="plan", refresh=true)` once and verify changed fragments
appear under `invalidated`. Do not refresh on every turn: normal reads use the
validated in-process file index.

When checking step 5, accept `exact_field_contract` only as a strict payload
whitelist. A `runtime_field_guide` is machine-readable discovery metadata for a
compatibility action; the server's action-specific validator remains
authoritative.
