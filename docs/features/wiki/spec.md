# Design: Wiki Mode

> 對應 index.md（原 manpage）「wiki mode」：知識庫模式，NotebookLM 風格。
> Agent: **librarian**

## 定位

- 像 **NotebookLM + llm-wiki** 的知識庫
- 使用者**上傳任意文件**，LLM 自動協助整理
- 文件解析：anydoc / markitdown / OCR
- RAG：是候選方案（`RAG ?` 待定）

## Agent 設計

### librarian

- 負責知識庫的**攝取、整理、查詢**
- **核心工具**：
  - `ingest_document(path|url)` — 文件解析 + 入庫
  - `kb_search(query)` — 知識庫檢索
  - `kb_summarize(topic)` — 生成主題摘要
  - `kb_link(topic_a, topic_b)` — 建立概念關聯
- **可以被 call**：normal mode 的 orchestrator、researcher（添加資料請 librarian 更新）、任何其他 agent

## 文件處理 Pipeline

```
上傳文件 → 解析 → 分塊 → 索引 → 組織
          (anydoc/     (embedding (vector     (LLM 整理:
           markitdown/  + 存儲)    DB)         wiki 頁面/關聯)
           OCR)
```

### 解析層（複用現成）

| 格式 | 工具 | 備註 |
|---|---|---|
| Office / PDF / HTML / EPUB | **markitdown**（Python，20+ 格式）| CLI + lib + MCP server |
| Word/PPT/Excel/PDF → GFM | **anydoc**（Rust，<5ms）| 無 OCR |
| 掃描 PDF | **markitdown-ocr**（LLM-vision plugin）| 需視覺模型 |
| arXiv | **pi-arxivist**（pandoc WASM，math 保留）| 論文專用 |
| YouTube / 影片 | **pi-web-access** 的 `fetch_content`（Gemini multimodal）| 影片摘要 |

### 索引層（選擇題）

| 方案 | 特色 | 取捨 |
|---|---|---|
| **pi-knowledge** | Local-first RAG：BM25 + vectors + rerank + code-aware chunking；零 API key embedding | ✅ 最完整現成；`~/.pi/knowledge/` |
| **pi-hermes-memory** | HERMES 移植：L1-L4 分層 + FTS5 session search | 只有 FTS5，無 vector |
| **pi-memory + qmd** | 最 popular：daily logs + keyword/semantic/deep 三模式 | 單一 memory 概念，非多 KB |
| **pi-semantic-memory** | LogosDB MCP：local vector search | stdio-only、每 turn 自動注入 |
| **自建 RAG**（pgvector/Qdrant）| 完全掌控 | 工作量大 |

**建議**：v1 用 **pi-knowledge**（最完整的現成 RAG），若要 notebookLM-style 的互連 wiki 再加 **llm-wiki 的三層模型**（raw sources / LLM-generated wiki MD / schema）當組織層。

## 知識庫模型

- **source**：原始文件（檔案 + 解析後 markdown）
- **wiki page**：LLM 生成的概念頁面（llm-wiki 風格：概念互連 + backlinks）
- **chunk**：可分塊檢索的單位（source 的子集，帶 metadata）
- **embedding**：向量（供 semantic search）
- **關聯**：概念 ↔ 概念、概念 ↔ source、source ↔ source（引用）

## UI 需求

- **上傳區**：drag & drop / URL / arXiv ID / YouTube
- **知識庫瀏覽**：wiki 頁面 + 概念圖（graph view，如 Obsidian/llm-wiki）
- **搜尋**：keyword + semantic（顯示來源引用）
- **整理狀態**：哪些文件已入庫、哪些在排隊、LLM 整理進度
- **閱讀視角**：source viewer（原始文件）+ wiki 視角（整理後）並排

## 整合點

- **Storage**：blob（原始檔）+ vector DB + Postgres（metadata）
- **Session**：知識庫 per-user、可多本（notebook 模型），可掛到 project/workspace 使用；跨 session 持久
- **Memory**：librarian 的整理結果進長期記憶
- **其他 agent**：researcher / writer 查知識庫 → `kb_search` tool

## 開放問題

1. **RAG 還是 llm-wiki 式組織？**（或兩者都做：RAG 檢索 + wiki 組織）
2. ✅ 已解決（2026-08-16）：**一人多本 wiki/notebook**（Gemini Notebook 模型）；多人共用之後思考（Gemini Notebook 2026 更新已加共用功能，可參考其語意）
3. OCR 是否 v1 就要（掃描 PDF）？
4. 文件版本管理：更新文件後，相關 wiki 頁面要自動重寫嗎？
