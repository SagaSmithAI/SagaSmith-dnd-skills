# SagaSmith D&D Skills

跨平台 D&D 5e 2014/2024 Agent Skills。

完整模式需要安装 `sagasmith-core` 和 `sagasmith-dnd`。nanobot、Codex、
Claude、OpenClaw 等平台均通过 Skill 调用同一 `sagasmith-dnd --json` CLI，
本仓库不包含数据库、向量、规则引擎或平台专属 Tools。

未安装 Runtime 时可使用 Portable 模式查阅随附 SRD，但不能持久化战役。

规则语料：

- SRD 5.2.1 / 2024 英文
- SRD 5.1 / 2014 英文
- SRD 5.1 / 2014 中文便利译本

模组生成由独立的 `SagaSmith-module-gen-skills` 维护；其现有 Markdown 输出可
直接交给 `sagasmith-dnd module ingest`。
