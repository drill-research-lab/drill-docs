# Writing（名稱暫定）— Track B（DSH-based）註記

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/dsh.md](../../tracks/dsh.md)。
> 深度調查：[reference/dsh.md](../../reference/dsh.md)。

## seam / plugin 對應

| spec 需求 | DSH 對應 |
|---|---|
| writer agent | custom preset（copy `standard`，加上學術工具面）|
| LaTeX compile | 沙盒內 tectonic/latexmk（`ctx.fs` + sandbox）；無現成 CLSI 對應——自建 compile service plugin |
| 學術工具鏈 | pi 生態 plugins 多為 CLI/MCP 形態，可經 MCP（`dsh-mcp-client`）或子進程接入——軌道無關 |
| deliverables | web UI 有 `ui-deliverables` plugin（產出物面板，PDF 可掛）|

## github issue

- 標題：`paper: 類 Overleaf 的寫作環境（Writing）— LaTeX/PDF 並排編輯、模板、自動編譯閉環、agent 協作`
- 連結：https://github.com/drill-research-lab/deepseek-harness/issues/25

### 內文（草稿，可直接貼至 issue）

```markdown
## 目標

實作一個具備 **Overleaf + agent**（vscode 編輯 LaTeX + 預覽 + copilot）使用者體驗的寫作功能（Drill 產品名：**Writing**，名稱暫定）。

核心閉環：**從素材或需求出發 → 與 agent 協作編寫 LaTeX → 自動編譯並回饋錯誤 → 產出可預覽、可下載的 PDF 報告**。撰寫、編譯與預覽全程在同一環境完成。

## 需求

### 1. 報告列表入口

- 主畫面 UI 左側、Chat 下方有一個區塊，列出所有報告
- 支援多份報告並存（每份獨立），可建立與切換

### 2. 編輯環境（類 Overleaf）

- 主編輯畫面：左側 LaTeX 原始碼、右側 PDF 預覽（類 Overleaf 佈局）
- 與 agent 的對話區預設在畫面下方（可依 UX 調整）
- 使用者可自行編輯內容，agent 也可修改——協作編輯，體驗像 vscode + copilot

### 3. 模板

- 內建多個報告模板（常見學術 / 報告格式）
- 支援上傳自訂模板

### 4. 起始來源

使用者可從多種起點開始一份報告：

- 帶入固定內容資源（既有檔案）
- 從知識庫（wiki / Library）取用素材
- 從 prompt / 需求描述出發
- 或由 general agent（Chat 入口的 agent）訂定需求，從頭到尾完成一份報告

### 5. 自動編譯與錯誤回饋（核心loop）

- 每次 agent 結束一輪輸出後，自動編譯 LaTeX
- 編譯錯誤資訊與 LSP 診斷資訊回傳給 agent，讓 agent 持續修正
- 迭代直到報告完成且可成功 compile

### 6. Agent 調用

- 支援某種形式讓 DSH 原本的 chat / agent（general agent）能夠調用此寫作能力（writer），以端到端完成一份報告

### 7. 交付出口（預留接口，非本 issue 驗收範圍）

- 完成的報告（PDF / LaTeX source / 摘要）應可**送入知識庫（wiki / Library）**成為可重用研究資料
- 本 issue 只需保證出口在架構上存在（產出物可被程式化取用／發布）；實際接線屬後續整合 issue

## 驗收條件（建議）

- [ ] 左側 Chat 下方可見報告列表，可建立多份並切換
- [ ] 編輯畫面為 LaTeX / PDF 並排，對話區在下方
- [ ] 可從內建模板建立報告，也可上傳自訂模板
- [ ] 可帶入既有資源（檔案 / 知識庫素材 / prompt）作為起始內容
- [ ] agent 一輪輸出後自動重新編譯；編譯錯誤與 LSP 診斷會回傳給 agent 並被修正
- [ ] general agent 能以某種形式調用寫作能力，從需求到成功 compile 的 PDF 一氣呵成

## 範圍備註

本 issue 聚焦上述寫作閉環。多人即時協同編輯、markdown-only / pandoc 路線、引用與書目管理（Zotero / BibTeX / CSL）、報告產生器（template + skills 組合）細節，屬 spec 開放問題或另案處理，見 Writing spec。

## 實作偏好

- 推薦以 **dsh-plugin** 方式完成；非必要不要修改 DSH 核心程式碼
- 優先參考或組合既有 dsh-plugins 以及其他開源實作

## 參考專案與連結

**UX / 產品**

- Overleaf（UX 基準：線上 LaTeX 協作編輯 + 即時 PDF 預覽；模板庫）：https://www.overleaf.com/
- Overleaf CE（開源版；AGPL-3.0，需四個服務，整合成本較高）：https://github.com/overleaf/overleaf
- LaTeX Workshop（VS Code 的 LaTeX 編輯 + 預覽 + 診斷——「vscode 編輯 latex + 預覽」的成熟參考）：https://github.com/James-Yu/LaTeX-Workshop
- texlab（LaTeX Language Server——LSP 診斷的現成實作）：https://github.com/latex-lsp/texlab
- SwiftLaTeX（輕量開源替代：瀏覽器端 WASM LaTeX 編譯）：https://github.com/SwiftLaTeX/SwiftLaTeX
- tectonic（自包含 TeX 引擎，自建 compile service 常用）：https://github.com/tectonic-typesetting/tectonic
- latexmk（LaTeX 編譯排程器）：https://ctan.org/pkg/latexmk

**Drill 規格文件**

- Writing spec（共同契約；含編輯器選擇題、編譯服務與報告產生器定位）：https://github.com/drill-research-lab/drill-docs/blob/main/docs/features/paper/spec.md
- Track B 註記：https://github.com/drill-research-lab/drill-docs/blob/main/docs/features/paper/dsh.md

**DSH plugins 生態**

- https://github.com/awesome-dsh-plugin/awesome-dsh-plugin
- https://github.com/topics/dsh-plugin
```
