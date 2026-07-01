# SagaSmith D&D CLI Contract

- Executable: `sagasmith-dnd`
- Agent calls always include `--json`.
- stdout contains exactly one JSON document.
- stderr may contain logs or model download progress.

Success:

```json
{"ok":true,"data":{},"error":null,"meta":{"command":"group.action","version":"0.2.0"}}
```

Failure:

```json
{"ok":false,"data":null,"error":{"code":"not_found","message":"..."},"meta":{}}
```

Exit codes: 0 success, 2 invalid input, 3 not found, 4 conflict, 5 configuration,
6 optional dependency, 10 internal error.

Keep responses bounded with `--limit`. Search first and expand one selected chunk.

Compatibility workflows include:

- `module index` / `module export-scenes`
- `save regenerate-recap`
- `memory scope` / `memory status`
- `state history` / `state undo` / `state redo`
