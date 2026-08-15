# Design: Paper Mode

> 對應 manpage.md「paper mode」：與 agents 一起建立 paper。
> Agent: **writer**

## 定位

- 跟 agents 一起建立 paper（**md or latex**）
- 自帶 **latex builder / preview / pdf viewer**
- 像 **Overleaf** 一樣

## Agent 設計

### writer

- 負責**論文撰寫、LaTeX 生成、文獻引用**
- **核心工具**：
  - `write_section(topic, outline)` — 寫章節
  - `rewrite_section(text, instruction)` — 改寫（anti-AI voice）
  - `add_citations(claims)` — 引用查證 + 插入 citation
  - `verify_citations(bib)` — 驗證 BibTeX（pi-bib 模式）
  - `compile_latex(project)` — LaTeX 編譯 + PDF
  - `format_references(style)` — 引用格式轉換（APA/MLA/Vancouver…）
- **可以被 call**：normal 的 orchestrator（「把實驗結果寫成論文段落」）、researcher（「這是我的實驗數據，幫我寫進 methods」）

## 學術 Pipeline（參考 pi-paper-lab / portos-wang）

```
研究問題 → 文獻搜尋 → 論文結構規劃 → 分章撰寫 → 引用查證 → 編譯 → 審查迭代
          (Semantic      (/ars-plan)    (writer)   (pi-bib)   (latex)  (review mode)
           Scholar/
           arXiv/
           OpenAlex)
```

## 撰寫工具鏈（複用現成）

| 功能 | 工具 | 備註 |
|---|---|---|
| **文獻搜尋** | hinsencamp/pi-research-agent（SS+arXiv+OpenAlex，免 key）| 搜尋+摘要+ranking |
| **論文生成** | Aspis0/pi-paper-lab（anti-AI rewrite + citations + .docx）| Vancouver + Word-native citations |
| **引用驗證** | pi-bib（CrossRef + Semantic Scholar 檢查 BibTeX）| DOI-first |
| **全文抓取** | pi-arxivist（pandoc WASM，math 保留）/ @wienerberliner/pi-arxiv | LaTeX source → MD |
| **Zotero** | paper-pilot MCP（6 DB + PDF 閱讀 + Zotero sync）/ academic-agent | 文獻庫管理 |
| **完整學術套件** | portos-wang/pi-extensions（39-agent：Deep Research 13 / Paper 12 / Reviewer 7 / Pipeline 10）| 可整包參考 |
| **LaTeX 編輯** | Overleaf CE（自架）或自建 | 見下 |

## LaTeX 編輯器（選擇題）

| 方案 | 說明 | 取捨 |
|---|---|---|
| **直接整合開源方案** | Overleaf CE 或更輕的開源 LaTeX editor（政策見 decisions.md D7：有開源替代直接用/整合/重寫） | 先盤點候選；Overleaf CE 需四服務（realtime + updater + docstore + history）偏重 |
| **自建 Yjs-based editor** | CodeMirror 6 + Yjs 協同 + 自建 compile service（tectonic / latexmk in Docker）| 輕量、可掌控、較多工作 |
| **只用 markdown + pandoc** | 不即時協同，寫完轉 PDF | 最簡；但不像 overleaf |

**建議**：v1 用 **markdown + pandoc/tectonic**（單人寫作 + 編譯），協同編輯留 v2（屆時 Yjs）。

## LaTeX Compile Service（若自建）

- **tectonic**（單一二進位、無 TeX 發行版依賴）或 **latexmk in Docker**
- 每次 compile 在容器隔離執行（Overleaf CLSI 模式）
- PDF cache + incremental compile（只重編變更檔案）
- **Preview**：PDF.js / KaTeX（單段即時 preview）

## UI 需求

- **編輯器**：Markdown 或 LaTeX（CodeMirror 6，syntax highlight）
- **PDF viewer**：並排（左編輯右 PDF，Overleaf 風格）
- **引用管理**：citation palette（從知識庫/Zotero 插入 `\cite{key}`）
- **大綱**：章節樹狀結構
- **模板**：內建（NeurIPS / ICML / IMRAD / 國科會計畫…）
- **編譯狀態**：compile 進度、錯誤訊息（log 位置）

## 整合點

- **Wiki**：文獻/引用來源 → librarian 管理
- **Research**：實驗結果 → researcher 產出 → writer 引用
- **Review**：初稿 → review mode 送審
- **Knowledge base**：citation lookup（Zotero / 知識庫）

## 🥇 第一個垂直切片：國科會 AI Slop 報告產生器

> **定位**：不是獨立專案——是 paper mode 的 **template + skills** 組合，也是 Drill 第一個實際交付給老紀（教授）的功能。

### 為什麼是這個

- 老紀要的第一個具體東西（Discord 討論 2026-08-12：「先開一個 Loop 做老紀想要的國科會 AI slop 報告產生器」）
- 它驗證的其實是 **paper mode 本身**：template 系統 + writer agent + citation + LaTeX compile
- 做完國科會報告產生器 = paper mode 的核心已可運作，其他模板（NeurIPS/ICML）只是換 template

### 組成

| 元件 | 內容 |
|---|---|
| **template** | 國科會計畫書 LaTeX 模板（章節結構：摘要/背景/方法/預期成果/經費…）|
| **skill** | `nstc-proposal` skill——教 writer 怎麼寫國科會風格（格式、口吻、份量）|
| **writer agent** | 用上面 template + skill 產出草稿 |
| **librarian** | 抓相關文獻/引用 |
| **reviewer** | 驗證引用真實性 + 內容品質 |
| **compile** | LaTeX → PDF（tectonic/latexmk in Docker）|

### 驗證範圍（完成 = paper mode 核心可用）

- [ ] template 系統（任意 LaTeX 模板可載入）
- [ ] writer agent + skill 協作
- [ ] citation 從知識庫插入
- [ ] LaTeX compile → PDF preview
- [ ] 國科會格式的實際產出可交付

### 開放問題

1. 國科會計畫書有官方 LaTeX 模板嗎？還是自建？（可問老紀/阿柏）
2. 要不要先從「單人寫作 + markdown → pandoc」最快路徑開始，LaTeX 後補？
3. skill 是預先寫好的 `nstc-proposal`，還是讓 agent 從範例文件學習？

## 開放問題

1. v1 用 markdown-only 還是 md + latex 並行？
2. ✅ 已解決：國科會模板 = 第一個垂直切片（見上方）
3. 協同編輯（多人同時寫）是 v1 需求嗎？
4. 引用格式：BibTeX 為主還是支援 CSL（Citation Style Language）？
