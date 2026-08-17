# Design: Wiki Mode

> 對應 index.md（原 manpage）「wiki mode」：知識庫模式，NotebookLM 風格。
> Agent: **librarian**

## 定位

- 像 **NotebookLM + llm-wiki** 的知識庫
- 使用者**上傳任意文件**，LLM 自動協助整理
- 文件解析：常見文書格式 → Markdown（含 OCR 選配）
- RAG：是候選方案（`RAG ?` 待定）

## Agent 設計

### OpenCode / oh-my-openagent 設計參考

| 生態 | 參考 agent | 可借設計 | 匹配邊界 |
|---|---|---|---|
| OpenCode | `scout` / `explore`（subagent） | `scout` 做外部文件與 dependency 調查；`explore` 以 read/grep/glob/web 工具做唯讀搜尋 | 只對應「查找」與唯讀權限；沒有攝取、索引或 wiki 整理能力 |
| oh-my-openagent | `Librarian`（subagent） | 文件查找、多 repo 分析、唯讀且不可 write/edit/delegate；由 caller 按需派遣 | 名稱與 specialist 形態直接對應，但其資料源是外部 docs/code，不是持久知識庫 |

Drill librarian 在上述查找 specialist 之上增加 `ingest_document`、索引、持久化與 wiki 組織。DeepWiki 查證位置：OpenCode `packages/opencode/src/agent/agent.ts`；omo Librarian agent 定義與 `docs/guide/orchestration.md`。完整 roster → [reference/domains.md](../../reference/domains.md) §1。

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
           (文書格式→MD  (embedding (檢索引擎    (LLM 整理:
            含 OCR 選配)  + 存儲)    見軌道檔)    wiki 頁面/關聯)
```

### 解析層（需求；工具選型見軌道檔）

| 格式需求 | 備註 |
|---|---|
| Office / PDF / HTML / EPUB（20+ 常見格式）→ Markdown | 必備 |
| 掃描 PDF（OCR）| 選配（需視覺模型）——開放問題 #3 |
| arXiv / LaTeX source | 論文專用，math 要保留 |
| YouTube / 影片 | 影音摘要（multimodal）|

> 通用工具調查（markitdown / anydoc / OCR 方案）→ [reference/domains.md](../../reference/domains.md) §4；各軌道選型 → [pi.md](pi.md) / [dsh.md](dsh.md)。

### 索引層（選型題，見軌道檔）

檢索方案的選擇（local RAG 引擎、FTS5、vector DB、自建）與「RAG vs llm-wiki 式組織」的取捨——**Track A 選型 → [pi.md](pi.md)**；DSH 線對應 → [dsh.md](dsh.md)。

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
