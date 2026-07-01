# 模组运行结构

- 静态章节、场景、房间和检索向量不进入存档。
- 当前场景、房间、探索度、线索和世界变化进入存档。
- 章节转换前记录事件并创建快照。
- 恢复快照会创建分支，不改变模组静态原文。

```powershell
sagasmith-dnd event add --campaign <id> --type chapter --summary "<transition>" --json
sagasmith-dnd save create --campaign <id> --label "<chapter transition>" --json
```
