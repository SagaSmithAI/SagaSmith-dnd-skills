# 🐉 SagaSmith D&D Skills

[中文](README.md)

SagaSmith D&D Skills 提供两种使用方式，Agent 请根据环境**自动选择**。

---

## 📦 full/ — 完整版（推荐）

需要本机安装 `sagasmith-dnd` Python 包。

```bash
pip install "sagasmith-dnd[all]"
sagasmith-dnd doctor --json
```

加载 `full/SKILL.md`。

优势：SQLite 持久化、FTS5 全文检索、ChromaDB 语义搜索、PDF 导入、Snapshot DAG。

---

## 🪶 standalone/ — 轻量版

无需安装 Python 包。Python 标准库即可。

加载 `standalone/SKILL.md` 并确保 `standalone/portable.py` 可用。

数据存 `~/.sagasmith/`。纯文件系统，零依赖。
不支持 PDF 导入、FTS5、ChromaDB。

---

## 许可

MIT
