# 角色创建

1. 先读取战役 edition。
2. 根据该 edition 的 SRD 指导种族/物种、背景、职业、属性和装备。
3. 展示完整草稿并等待玩家确认。
4. 确认后才写入 Runtime。

```powershell
sagasmith-dnd character create --campaign <id> --name "<name>" --type pc --player "<player>" --sheet '<json>' --json
sagasmith-dnd character list --campaign <id> --json
sagasmith-dnd character show --id <character-id> --json
```

角色 sheet 至少包含 abilities、level、armor_class、hit_points、
max_hit_points、class、species、background、proficiencies、inventory 和 spells。
