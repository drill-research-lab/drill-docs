# Library — Track B（DSH-based）註記

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/dsh.md](../../tracks/dsh.md)。
> 深度調查：[reference/dsh.md](../../reference/dsh.md)。

## seam / plugin 對應

| spec 需求 | DSH 對應 |
|---|---|
| 文件儲存 | `ctx.fs` seam（`fs-local` provider，可換 sandboxed/workspace 化的 provider）|
| 檢索/索引 | 無內建 RAG primitive——需自建 plugin（或經 MCP 接 pi-knowledge 等外部引擎）|
| 文件解析 | markitdown/anydoc 為獨立工具（語言中立），以子進程或 MCP 接入 |
| KB MCP | 經 `dsh-mcp-client` 消費外部 KB MCP server |

## github issue

- 標題：`wiki: 類 NotebookLM / llm-wiki 的知識庫（Library）— 上傳轉 Markdown、資源預覽、agent 問答`
- 連結：https://github.com/drill-research-lab/deepseek-harness/issues/23

### 內文（草稿，可直接貼至 issue）

```markdown
## 目標

實作一個具備 **(Gemini) NotebookLM / llm-wiki** 使用者體驗的知識庫功能（Drill 產品名：**Library**）。

核心閉環：**上傳檔案 → 轉為 Markdown 保存 → 人可瀏覽預覽、agent 可閱讀回答**。研究資料上傳一次，人與 agent 共用。

## 需求

### 1. 知識庫列表入口

- 主畫面 UI 左側、Chat 下方有一個區塊，列出所有知識庫
- 支援多本知識庫並存（每本獨立），可建立與切換

### 2. 上傳與文件轉換

- 可上傳任何常見格式的檔案（Office / PDF / HTML / EPUB …）
- 上傳後系統將檔案轉為 **Markdown**，保存在工作目錄中，供 agent 閱讀與後續檢索
  - 解析方式不預設單一工具；候選：anydoc / markitdown / OCR / 既有 API
- **原始檔案保留**，供預覽與下載

### 3. 資源瀏覽與預覽

- 使用者可以看到知識庫內所有資源
- 支援 **Markdown 預覽**、**PDF 預覽** 等
- 上傳與瀏覽全程不需要進入聊天

### 4. Agent 存取（librarian）

- agents 能閱讀這些已轉換的文檔並回答問題
- DSH 原本的 chat / agent 可以透過某種方式存取 **librarian**（知識庫查詢能力）
- 工具面可參考 DeepWiki MCP 的三個工具設計——
  - `ask_question`（**核心**）：對知識庫直接提問，取得有上下文根據、附引用的回答——agent 使用知識庫的主要互動方式是「問」，不是逐檔翻找
  - `read_wiki_structure`（結構導覽）：知道知識庫裡有什麼、分哪些主題
  - `read_wiki_contents`（內容讀取）：需要原文時整份讀取
  - 也就是：小而穩定的 agent 查詢介面，問答為主、結構與內容讀取為輔

### 5. 內容入口（預留接口，非本 issue 驗收範圍）

- 除 UI 上傳外，知識庫應預留**程式化寫入入口**，供外部系統將內容送入：pipeline 的 sources / results、完成的報告（Writing deliverable）、agent 整理的文件等
- 入口須保留內容類型標記（source / result / deliverable），對應 spec 對「原始來源」與「綜整結果」的區分要求
- 本 issue 只需保證此入口在架構上存在（介面與資料模型不封死）；實際接線（如 pipeline destination 自動流入）屬後續整合 issue

## 驗收條件

- [ ] 左側 Chat 下方可見知識庫列表，可建立多本並切換
- [ ] 上傳檔案後，工作目錄出現轉換後的 Markdown，且原始檔保留
- [ ] 資源清單可直接預覽 Markdown 與 PDF
- [ ] chat / agent 能經 librarian 對知識庫**提問**，取得有根據的回答；也能查結構、讀內容

## 範圍備註

本 issue 聚焦上述基礎閉環。llm-wiki 式概念頁面組織、檢索引擎選型（RAG vs wiki 組織）、Scheduled Search 結果匯入論文庫等，屬 spec 開放問題，後續另開 issue。

## 實作偏好

- 推薦以 **dsh-plugin** 方式完成；非必要不要修改 DSH 核心程式碼
- 優先參考或組合既有 dsh-plugins 以及其他實作

## 參考專案與連結

**UX / 產品**

- Gemini Notebook（原 NotebookLM，2026-07 改名；sources + grounded Q&A + inline citations）：https://notebooklm.google/ · 說明文件：https://support.google.com/gemininotebook
- Karpathy — LLM Wiki pattern（原始 gist；ingest 時編譯成持久互連的 wiki，而非 query-time RAG）：https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- llm-wiki 實作：https://github.com/nvk/llm-wiki · https://github.com/cobusgreyling/llm-wiki · https://github.com/microsoft/llmwiki
- DeepWiki MCP（agent 消費文件知識庫的極簡工具面參考；MCP 三工具：`ask_question` 問答為核心 + `read_wiki_structure` / `read_wiki_contents`）：https://deepwiki.com/

**Drill 規格文件**

- Library spec（共同契約）：https://github.com/drill-research-lab/drill-docs/blob/main/docs/features/wiki/spec.md
- Track B 註記：https://github.com/drill-research-lab/drill-docs/blob/main/docs/features/wiki/dsh.md

**DSH plugins 生態**

- https://github.com/awesome-dsh-plugin/awesome-dsh-plugin
- https://github.com/topics/dsh-plugin
```
