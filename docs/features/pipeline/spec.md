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

## UI 需求

- **視覺化 DAG editor**（拖放節點、連線、設定）
- 節點 inspector（設定 prompt / skills / tools / code）
- cron 管理介面（排程列表 + 下次執行時間）
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
