# Design: Paper Mode

> 對應 index.md（原 manpage）「paper mode」：與 agents 一起建立 paper。
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
  - `verify_citations(bib)` — 驗證 BibTeX（DOI-first 查證）
  - `compile_latex(project)` — LaTeX 編譯 + PDF
  - `format_references(style)` — 引用格式轉換（APA/MLA/Vancouver…）
- **可以被 call**：normal 的 orchestrator（「把實驗結果寫成論文段落」）、researcher（「這是我的實驗數據，幫我寫進 methods」）

## 學術 Pipeline

```
研究問題 → 文獻搜尋 → 論文結構規劃 → 分章撰寫 → 引用查證 → 編譯 → 審查迭代
           (學術 DB      (規劃)        (writer)  (DOI 查證) (latex) (review mode)
            搜尋引擎)
```

> 工具鏈選型（文獻搜尋 / 論文生成 / 引用驗證 / 全文抓取 / Zotero 的現成工具）→ [pi.md](pi.md)（Track A）；通用調查 → [reference/pi-ecosystem.md](../../reference/pi-ecosystem.md) 學術工具節。

## LaTeX 編輯器（選擇題）

| 方案 | 說明 | 取捨 |
|---|---|---|
| **直接整合開源方案** | Overleaf CE 或更輕的開源 LaTeX editor（政策見 [decisions.md](../../decisions.md) D7：有開源替代直接用/整合/重寫） | 先盤點候選；Overleaf CE 需四服務（realtime + updater + docstore + history）偏重 |
| **自建 Yjs-based editor** | CodeMirror 6 + Yjs 協同 + 自建 compile service（tectonic / latexmk in Docker）| 輕量、可掌控、較多工作 |
| **只用 markdown + pandoc** | 不即時協同，寫完轉 PDF | 最簡；但不像 overleaf |

> 此選擇題兩軌共通（編輯器是前端+compile service 問題，與 harness 無關）——暫留 spec，定案後移出。

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

## 報告產生器（paper mode 的模板應用）

> **定位**：報告產生器 = paper mode 的 **template + skills** 組合能力，不是獨立專案。
> 模板只是參數：國科會計畫書、NeurIPS/ICML、IMRAD、技術報告……換模板即換產出，**國科會只是其中一個模板**（緣起：老紀要的第一個具體東西，Discord 2026-08-12「先開一個 Loop 做老紀想要的國科會 AI slop 報告產生器」）。

### 組成

| 元件 | 內容 |
|---|---|
| **template** | 目標格式的模板（例：國科會計畫書章節結構——摘要/背景/方法/預期成果/經費…）|
| **skill** | 對應風格 skill（例：`nstc-proposal`——教 writer 該格式的口吻、份量、慣例）|
| **writer agent** | 用 template + skill 產出草稿 |
| **librarian** | 抓相關文獻/引用 |
| **reviewer** | 驗證引用真實性 + 內容品質 |
| **compile** | LaTeX → PDF（tectonic/latexmk in Docker）|

### 驗證範圍（通用，不綁特定模板）

- [ ] template 系統（任意 LaTeX/MD 模板可載入）
- [ ] writer agent + skill 協作
- [ ] citation 從知識庫插入
- [ ] LaTeX compile → PDF preview
- [ ] 任一模板（如國科會）的實際產出可交付

### 開放問題

1. 國科會計畫書有官方 LaTeX 模板嗎？還是自建？（可問老紀/阿柏）
2. 要不要先從「單人寫作 + markdown → pandoc」最快路徑開始，LaTeX 後補？
3. skill 是預先寫好（如 `nstc-proposal`），還是讓 agent 從範例文件學習？

## 開放問題

1. v1 用 markdown-only 還是 md + latex 並行？
2. ✅ 已解決：報告產生器＝paper mode 的 template+skills 組合，模板可置換（國科會為其一）
3. 協同編輯（多人同時寫）是 v1 需求嗎？
4. 引用格式：BibTeX 為主還是支援 CSL（Citation Style Language）？
