# Pipelines — Track B（DSH-based）註記

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/dsh.md](../../tracks/dsh.md)。
> 深度調查：[reference/dsh.md](../../reference/dsh.md)。

## 實作選項（自 spec 移入）

| 選項 | 說明 | 取捨 |
|---|---|---|
| **DSH schedule seam** | `ctx.schedule`（session-local scheduled follow-ups；`packages/schedule/schedule/`） | 排程 follow-up 用；**不是 full workflow engine**——pipeline 本體仍要建 |
| **DSH 內建 workflow 面板** | web UI 有 `ui-workflow-run` feature plugin | 執行顯示可參考；DAG editor 仍需自建 |

DSH `standard` preset 內建 `workflow` tool（e2e 實測 23 tools 之一）——命名相近，實際能力待查（可能已含 workflow 執行原語，若是則 Track B 的 pipeline 引擎可大幅省工）。

## seam / plugin 對應

| spec 需求 | DSH 對應 |
|---|---|
| cron trigger | `ctx.schedule` seam + 持久化 job registry（要自補）|
| MCP toolcall trigger | `dsh-mcp-client` 消費側；expose 側待建 |
| 節點三層（llm/agent/code） | agent presets（`minimal` 節點型 / `standard` builder）；code 節點 = sandbox |
| 執行狀態 resume | append-only session log + `ctx.sessionQuery` |

## Scheduled Search template（tracer bullet）

對應 [spec.md](spec.md) 的第一個 built-in template：

| Template 元件 | Track B 對應 |
|---|---|
| arXiv source（預設） | 需新增 arXiv source adapter（專用 API/MCP）；不以一般 Web 搜尋取代穩定 arXiv ID |
| 其他 Web sources | `ctx.web` seam；現有 providers：DeepSeek / Exa / Perplexity，model-facing tools 為 `web_search` / `web_fetch` |
| cron / background run | `ctx.schedule` 可參考，但其 session-local follow-up 不滿足需求；必須補持久化 job registry 與無前端 session worker |
| normalize / dedupe | code runtime node；優先鍵 arXiv ID → DOI → canonical URL |
| persistence / provenance | append-only run events + workspace artifact collection；保存 provider、URL、取得時間與 external ID |
| status | `ctx.sessionQuery` 查 run lineage；另提供 last/next run projection 給 UI |

第一條 e2e 只要求 arXiv；source adapter 介面保留 DSH web providers，後續可直接加入其他 web search tools。

## github issue

- 標題：`pipeline: 類 n8n 的自動化 workflow（Pipelines）— DAG 編排、cron 背景執行、template 與 agent 觸發`
- 連結：https://github.com/drill-research-lab/deepseek-harness/issues/24

### 內文（草稿，待收斂）

```markdown
## 目標

實作一個具備 **n8n** 使用者體驗的自動化 workflow 功能（Drill 產品名：**Pipelines**）。

核心閉環：**定義 pipeline（或從 template 建立）→ 排程 / 觸發執行 → 背景運行並保存結果 → 人與 agent 都能取用產出**。可重複的研究流程定義一次，持續累積資料。

## 需求

### 1. Pipeline 列表入口

- 主畫面 UI 左側、Chat 下方有一個區塊，列出所有 pipelines / templates / schedules
- 可建立、切換、編輯、刪除

### 2. Pipeline 定義與編輯

- Pipeline = 節點 + 連邊組成的 workflow（DAG）
- 節點類型三層：
  - **code**：執行程式碼（資料處理、正規化）
  - **agent**：multistep agent 執行（可設定 prompt / skills / tools）
  - **llm**：single LLM ask（最輕）
- 節點輸出 = 後續節點輸入；支援條件分支與平行 fan-out / fan-in
- 提供視覺化編輯（DAG editor）；也可由 agent 代建（見 #6）

### 3. 觸發方式

- **手動**執行
- **cron 排程**：設定執行頻率與時區
- **agent 觸發**：pipeline 可被其他 agent 以 toolcall 形式呼叫（like MCP）
- 三者共存，同一 pipeline 不限單一觸發方式

### 4. 背景執行契約（排程的核心需求）

- 排程保存後，**關閉頁面 / session 結束仍必須執行**（background scheduled job，不依賴使用者開著 UI）
- 每次 run 保存：狀態、開始／結束時間、輸入、輸出、錯誤
- 單一節點失敗不應抹除其他節點已取得的結果；支援 retry
- 執行狀態持久化，可檢視歷史；long-running 可 resume
- UI 顯示 pipeline 狀態（running / success / failed / paused）、last run / next run

### 5. Templates

- 內建 template gallery：從 template 一鍵建立 pipeline，不需從空白 DAG 開始
- 第一個 built-in template：**Scheduled Search（定時搜尋）**——tracer bullet
  - inputs：`query`（關鍵字）、`sources`（搜尋來源，第一條 e2e 以 arXiv 為主）、`schedule`（cron + 時區）、`max_results`、`destination`（結果目的地，預設目前 Project）
  - 流程：排程觸發 → 搜尋 → metadata 正規化 → 去重（arXiv ID → DOI → canonical URL）→ 保存結果與 provenance（來源、URL、取得時間、外部 ID）→ 更新 run history
  - **去重是硬需求**：重跑不得重複建立相同資料
- template 本身可被修改、另存

### 6. 與 agent 的整合

- pipeline 可 expose 成 tool，讓 DSH 原本的 chat / agent 直接呼叫
- **pipeline builder**：使用者可以用對話請 agent 建立 / 修改 pipeline（「幫我建一個每週搜尋 X 的排程」），人在編輯器檢視與修改
- pipeline 中的 agent 節點可設定 skills / tools

### 7. 輸出與重用（共用資料集）

- Pipeline 的來源（sources）與結果（results）保存下來，供後續研究**直接取用，不需要重新跑**
- `destination` 可指向知識庫（wiki / Library）：結果自動流入，且保留「原始來源」與「模型綜整結果」的區分
- 未設定 destination 時，結果留在目前 Project

## 驗收條件（建議）

Scheduled Search tracer bullet：

- [ ] 從 template 建立 Scheduled Search，不需從空白 DAG 開始
- [ ] 設定 query、arXiv、cron、timezone 後可保存
- [ ] UI 關閉後，後端仍按排程執行
- [ ] arXiv 結果正規化、去重並持久化（含 provenance）
- [ ] UI 可看到 last run、next run、執行狀態與新增結果
- [ ] 重跑相同搜尋不產生 duplicate records
- [ ] 結果可交給後續論文庫／viewer，而不需要重新搜尋

Pipeline 本體：

- [ ] 可建立、編輯 pipeline（至少 code / llm / agent 三類節點各可運作一次）
- [ ] pipeline 可被手動 / cron / agent toolcall 三種方式觸發
- [ ] run 歷史與每節點 input/output 可檢視

## 範圍備註

本 issue 聚焦 pipeline 引擎與第一個 template。DAG editor 的完整編輯體驗、更多 sources（Semantic Scholar / OpenAlex / web search）、pipeline 間組合等，後續另開 issue。

## 實作偏好（非硬性）

- 推薦以 **dsh-plugin** 方式完成；非必要不要修改 DSH 核心程式碼
- 已知可參考的 DSH 既有面：web UI 的 `ui-workflow-run` plugin（執行顯示）、`ctx.schedule`（session-local follow-up，**不足以**支撐背景排程——需補持久化 job registry）、`ctx.web` seam（搜尋 providers）
- 優先參考或組合既有 dsh-plugins

## 參考專案與連結

**UX / 產品**

- n8n（UX 基準：workflow 自動化、node-based editor、cron / webhook triggers、template library）：https://n8n.io/ · https://github.com/n8n-io/n8n
- Node-RED（flow-based programming 老祖宗；node editor 互動參考）：https://github.com/node-red/node-red
- Activepieces（開源 Zapier 替代；template / piece 生態參考）：https://github.com/activepieces/activepieces
- Huginn（agent-based 自動化經典；「排程代理程式監看世界」的概念原型）：https://github.com/huginn/huginn
- Zapier / Make（閉源商業產品；UX 對照組）

**Drill 規格文件**

- Pipelines spec（共同契約；含執行模型、Scheduled Search template、outputs reuse 完整定義）：https://github.com/drill-research-lab/drill-docs/blob/main/docs/features/pipeline/spec.md
- Track B 註記：https://github.com/drill-research-lab/drill-docs/blob/main/docs/features/pipeline/dsh.md
- 需求來源（定時搜尋 / 共用資料集）：https://github.com/drill-research-lab/drill-docs/blob/main/docs/requirements/2026-08-19-neokent-followup.md

**DSH plugins 生態**

- https://github.com/awesome-dsh-plugin/awesome-dsh-plugin
- https://github.com/topics/dsh-plugin
```
