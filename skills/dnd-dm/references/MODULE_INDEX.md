# 模组索引与运行

模组由 Runtime 从 PDF 或 Markdown 建立章节、场景、房间、页码和检索块。

```powershell
sagasmith-dnd module inspect --path "<path>" --json
sagasmith-dnd module ingest --campaign <id> --path "<path>" --json
sagasmith-dnd module list --campaign <id> --json
sagasmith-dnd module search --campaign <id> --query "<query>" --limit 5 --json
sagasmith-dnd module expand --chunk <chunk-id> --json
sagasmith-dnd module read-scene --campaign <id> --scene <scene-id> --json
```

运行时只读取当前场景。隐藏房间、反转、NPC 动机和附录内容不得提前泄露。

进度更新：

```powershell
sagasmith-dnd module set-progress --campaign <id> --scene <scene-id> --progress 50 --room "<room>" --state '<json>' --json
```
