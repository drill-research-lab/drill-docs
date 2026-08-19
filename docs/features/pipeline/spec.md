# Design: Pipeline Mode

> 對應 index.md（原 manpage）「pipeline mode」：建立 pipeline 與 cron job，n8n 風格。
> Agent: **pipeline builder / mini agent / simple llm**

## 定位

- 建立與管理 **pipeline（DAG workflow）與 cron job**
- 像 **n8n** 在做的事情
- 觸發方式：
  - **cron**（定期執行）
  - **toolcall 觸發**（被 agent 當 MCP tool 呼叫）
- 每個節點可以是：**程式碼 / agent / llm**
- 建立後可被 agent 呼叫（like MCP）或 cronjob 觸發

## Agent 設計

Roster：**pipeline builder**（互動入口）＋ 節點執行的 **mini agent / simple llm**（非對話 agent，是 pipeline 的節點型別）。

### OpenCode / oh-my-openagent 設計參考

| Drill 角色 | OpenCode 參考 | oh-my-openagent 參考 | 匹配邊界 |
|---|---|---|---|
| pipeline builder | `plan`（primary）：只能編輯 `.opencode/plans/*.md` 等 plan path | `Prometheus`（primary）：訪談後產出 `.omo/plans/*.md`，由 md-only hook 固定產物邊界 | 兩者都只直接證明「受限規劃 artifact」；`WorkflowJSON` 生成與 DAG 編輯仍是 Drill 自建 |
| pipeline runner | 無直接內建對應 | `Atlas`（primary）：讀取計畫、逐項 dispatch、驗收與更新進度 | 可借 plan-execution orchestrator pattern，不等同通用 workflow runtime |
| mini agent | `general`（subagent）：接受多步驟 delegated task | `Sisyphus-Junior`（subagent）：由 `task(category=...)` 路由，聚焦執行且不能再派工 | 對應單節點 executor；category 只決定 model / prompt tuning / permissions，不提供 DAG state |
| simple llm | 無直接 agent 對應 | 無直接 agent 對應 | 應是無 agent loop 的單次 model call |

DeepWiki 查證位置：OpenCode `packages/opencode/src/agent/agent.ts`；omo `docs/guide/orchestration.md`、Prometheus / Atlas agent 定義與 category routing。節點工具面另參考 DSH `minimal` preset → [dsh.md](dsh.md)；完整 roster → [reference/domains.md](../../reference/domains.md) §1。

### 節點三層複雜度

| 節點類型 | 行為 | 說明 |
|---|---|---|
| **simple llm** | single LLM ask，結果傳給後續節點 | 最輕 |
| **mini agent** | multistep 執行（設定 prompt / skills / tools），完成後傳給後續節點 | 中等 |
| **pipeline builder** | 完整 agent，可動態建立/修改 pipeline | 最重 |

### pipeline builder agent

- 負責**建立 / 編輯 / 刪除 pipeline**（create/edit/delete nodes、連邊、設 cron、綁 MCP）
- 本質是 mini-IDE，需要 tool schema：
  - `pipeline_create(definition)`
  - `pipeline_edit(node_id, params)`
  - `pipeline_connect(from, to)`
  - `pipeline_set_cron(id, schedule)`
  - `pipeline_expose_mcp(id)` — 讓 pipeline 可被其他 agent 當 tool 呼叫
  - `pipeline_run(id, inputs)` / `pipeline_status(id)`

## Pipeline 執行模型

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│  Node 1 │ ──► │  Node 2 │ ──► │  Node 3 │
│  (llm)  │     │ (agent) │     │ (code)  │
└─────────┘     └─────────┘     └─────────┘
     │               │               │
     ▼               ▼               ▼
  input        multistep         output
  (prompt)     (prompt+           (結果)
               skills+tools)
```

- **Data flow**：節點輸出 = 後續節點輸入（JSON）
- **Branching**：條件分支（if/else）
- **Parallel**：fan-out → fan-in（執行期決定目標數量；形態如 pi-subagents 的 `runs.all` / pi-dynamic-workflows——見 [pi.md](pi.md)）
- **Retry / Gate**：節點失敗重試；acceptance gate（結果驗證後才往下）
- **State**：pipeline 執行狀態持久化，可 resume（long-running research workflow）

## 實作選項

| 選項 | 說明 | 取捨 |
|---|---|---|
| **n8n embed** | `@n8n/workflow-sdk` programmatic build；`McpService` MCP server mode；cron/webhook trigger 內建 | 功能最全；但 Sustainable Use License（商用有門檻）、無現成純 library execute API、batch 非 stream |
| **DSH schedule seam** | session-local scheduled follow-ups | 不完整（不是 full workflow engine） |
| **自建 FSM + scheduler** | 在 pi-agent-core / Cordis 上自建 | 完全掌控；cron + webhook + MCP trigger + node types 都要自己寫 |
| **pi-dynamic-workflows** | fan-out 100s subagents + /deep-research | 學術 research 場景強；非通用 DAG |

**建議**：v1 自建簡化版（因為 research pipeline 的節點 = 我們的 agent/llm，綁定很深，n8n 抽象是通用 workflow 不相容）；但要**抄 n8n 的設計**：
- node types registry（trigger / code / agent / llm）
- `WorkflowJSON` schema（可 LLM 生成 + 驗證）
- cron + webhook + MCP toolcall 三種 trigger
- MCP server expose（pipeline 被其他 agent 當 tool）

## Cron 排程

- 用外部 scheduler（node-cron / BullMQ）+ 持久化 job registry
- 支援：cron expression、時區、失敗重試、執行歷史
- agent 可以 `pipeline_set_cron()` 動態建立排程

## Built-in Template：Scheduled Search（tracer bullet）

> 需求來源：[老記(neokent)追加需求原文（2026-08-19）](../../requirements/2026-08-19-neokent-followup.md)。

「定時搜尋」不是新 mode 或新 agent，而是第一個 **built-in pipeline template**。它用現有 cron、search tool、artifact storage 與執行記錄串出第一條端到端 tracer bullet。

### Template inputs

| 參數 | 說明 | tracer bullet 預設 |
|---|---|---|
| `query` | 關鍵字或搜尋式 | 必填 |
| `sources` | 一或多個 search source adapter | `arxiv` |
| `schedule` | cron expression + timezone | 必填 |
| `max_results` | 每次執行最多取回數量 | 可設定 |
| `destination` | 結果目的地；未設定時使用目前 Project | 選填，預設 Project |
| `dedupe_key` | 去重策略 | arXiv ID → DOI → canonical URL |

### 預設流程

```text
cron trigger
  → 依 sources 執行搜尋
  → normalize metadata
  → 以 arXiv ID / DOI / canonical URL 去重
  → 保存新的 result records 與檔案／來源連結
  → 更新 run history、last_run、next_run
```

### 搜尋來源

- **第一條 tracer bullet 以 arXiv 為主**：先驗證 metadata 搜尋、穩定 ID、去重、持久化與後續預覽鏈路
- `sources` 必須是可插拔 adapter；已有的 web search / fetch tools 可作其他來源
- 後續候選：Semantic Scholar、OpenAlex、一般 Web search；不是第一條 tracer bullet 的 blocking scope
- 一次執行可選多來源，但各 provider 的 rate limit、錯誤與 provenance 必須分開記錄

### 背景執行契約

- 排程保存後，即使使用者關閉頁面或 session 結束也必須執行
- 每次 run 保存狀態、開始／結束時間、來源、命中數、新增數、重複數與錯誤
- 單一來源失敗不應抹除其他來源已取得的結果
- 結果必須帶 provenance（來源 provider、原始 URL、取得時間、外部 ID）
- 重跑不得重複建立相同資料；既有結果可直接取用，完整規則見下方「Pipeline outputs reuse」

### Tracer bullet 驗收

- [ ] 從 template 建立 Scheduled Search，不需從空白 DAG 開始
- [ ] 設定 query、arXiv、cron、timezone 後可保存；destination 未設定時使用目前 Project
- [ ] UI 關閉後，後端仍按排程執行
- [ ] arXiv 結果正規化、去重並持久化
- [ ] UI 可看到 last run、next run、執行狀態與新增結果
- [ ] 重跑相同搜尋不產生 duplicate records
- [ ] 結果可交給後續論文庫／viewer，而不需要重新搜尋

## Pipeline outputs reuse

原需求稱「共用資料集」，目前定義為：**Pipeline 的來源與結果可保存、發布並被後續研究重用**，不限定為表格型 dataset。

### Output destination

- `destination` 是選填配置；未設定時，sources 與 results 預設保存到目前 Project
- Pipeline 執行完成後，使用者可手動將選定的 sources 或 results 加入 Workspace Wiki
- Pipeline / template 可預先指定一個 Wiki，讓每次執行後的 sources / results 自動流入該 Wiki
- 即使自動流入 Wiki，Project 仍保留該次 pipeline run 與結果紀錄；Wiki 是可重用的發布目的地，不取代 Project 執行紀錄

### 內容區分與重用

- **Source Markdown**：搜尋取得的原始來源或由原始檔轉換的內容
- **Result Markdown**：Pipeline 綜整、分析或生成的研究結果
- 加入 Wiki 後仍須保留 source / result 的區分，不能把模型產生的結論偽裝成原始來源
- Normal Chat 不直接混入整個 Wiki context；需要 Wiki 內容時，由 orchestrator 呼叫 librarian subagent 查詢並帶回相關內容

## UI 需求

- **視覺化 DAG editor**（拖放節點、連線、設定）
- 節點 inspector（設定 prompt / skills / tools / code）
- cron 管理介面（排程列表 + 下次執行時間）
- template gallery / create flow（第一個 template：Scheduled Search）
- 執行記錄（每個 node 的 input/output、耗時、成功/失敗）
- pipeline 狀態（running / success / failed / paused）

## 整合點

- **MCP**：pipeline 建立後自動產生 MCP tool schema（input/output 型別從節點推導）
- **Skills**：mini agent 節點可設定 skill
- **其他 agent**：agent 透過 MCP toolcall 觸發 pipeline
- **Session**：long-running pipeline 的執行狀態跨 session 持久

## 開放問題

1. v1 用 n8n（功能全；但 TypeORM+DI 重、無純 library 執行 API、batch 非 stream）還是自建簡化版（掌控但工作量）？（純技術取捨——license 已由 decisions.md D7 排除在考量外）
2. pipeline 被 agent 呼叫的路徑（未定）：a) MCP expose（每 pipeline 一個 server vs 統一動態註冊）——**內部** agents 以 MCP tool 形式呼叫；b) **subagent-like 派遣**（跟 librarian 等其他 agent 一樣）——可能更自然；兩者可並存
3. pipeline builder agent 的權限：可以改自己的定義嗎？還是只能建新的？
4. cron job 的失敗處理：retry 幾次？通知誰？
