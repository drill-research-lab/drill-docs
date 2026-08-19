# Library — Track A（PI-based）設計

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/pi.md](../../tracks/pi.md)。
> 原則：最大化複用 PI 生態系（[reference/pi-ecosystem.md](../../reference/pi-ecosystem.md)），gap 才自建。

## 解析層選型（自 spec 移入；通用工具調查 → [reference/domains.md](../../reference/domains.md) §4）

| 格式 | 工具 | 備註 |
|---|---|---|
| Office / PDF / HTML / EPUB | **markitdown**（Python，20+ 格式）| CLI + lib + MCP server；OCR 需 `markitdown-ocr` plugin |
| Word/PPT/Excel/PDF → GFM | **anydoc**（Rust，<5ms）| 無 OCR；Node (napi-rs) / Python (PyO3) bindings |
| 掃描 PDF | **markitdown-ocr**（LLM-vision plugin）| 需視覺模型 |
| arXiv | **pi-arxivist**（pandoc WASM，math 保留）/ @wienerberliner/pi-arxiv | 論文專用 |
| YouTube / 影片 | **pi-web-access** 的 `fetch_content`（Gemini multimodal）| 影片摘要 |

## 索引層選型（自 spec 移入）

| 方案 | 特色 | 取捨 |
|---|---|---|
| **pi-knowledge** | Local-first RAG：BM25 + vectors + weighted fusion + cross-encoder rerank；code-aware chunking；零 API key（local ONNX `multilingual-e5-small`） | ✅ 最完整現成；`~/.pi/knowledge/` |
| **pi-hermes-memory** | HERMES 移植：L1-L4 分層 + FTS5 session search | 只有 FTS5，無 vector |
| **pi-memory + qmd** | 最 popular：daily logs + keyword/semantic/deep 三模式 | 單一 memory 概念，非多 KB |
| **pi-semantic-memory** | LogosDB MCP：local vector search | stdio-only、每 turn 自動注入 |
| **自建 RAG**（pgvector/Qdrant）| 完全掌控 | 工作量大 |

**建議（草稿）**：以 **pi-knowledge** 為檢索引擎；若要 NotebookLM 式互連 wiki，加 **llm-wiki 三層模型**（raw sources / LLM-generated wiki MD / schema，見 [reference/domains.md](../../reference/domains.md) §6）當組織層。此選型與 Chat 記憶選型共用同一決策（見 [normal/pi.md](../normal/pi.md)）。

## 自建 gap 清單

1. **多本 notebook 的資料模型**（spec 已定：per-user 可多本）——pi-knowledge 是單一 KB（`~/.pi/knowledge/`），需 multi-instance 化或自建 KB registry
2. **組織層**（wiki 頁面 / 概念關聯 / backlinks）——pi 生態無現成對應 llm-wiki 組織層的 plugin；需自建（llm-wiki 本身是 Tauri 桌面 app，不可直接嵌入，抄三層模型）
3. `kb_link` / `kb_summarize` tools——librarian 的組織工具需自建（包在 pi extension）
4. 上傳 UI / graph view（前端）
