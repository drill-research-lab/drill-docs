# Pipelines — Track A（PI-based）設計

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/pi.md](../../tracks/pi.md)。
> 原則：最大化複用 PI 生態系（[reference/pi-ecosystem.md](../../reference/pi-ecosystem.md)），gap 才自建。

## 實作選項（自 spec 移入）

| 選項 | 說明 | 取捨 |
|---|---|---|
| **n8n embed** | `@n8n/workflow-sdk` programmatic build；`McpService` MCP server mode；cron/webhook trigger 內建 | 功能最全；但無純 library execute API（TypeORM+DI 重）、batch 非 stream——**結論：不 embed，抄設計**（spec 共同結論） |
| **pi-dynamic-workflows** | fan-out subagents（上限 16 併發/1000 每 run）+ model tier routing + journal resume + `/deep-research` | 學術 research 場景強；非通用 DAG——可當 fan-out 節點的執行核心參考 |
| **pi-subagents workflowScript** | `runs.run` / `runs.all` / branching / retry / gate / aggregation（JS） | 節點執行語意的好參考；不是持久化 pipeline 引擎 |
| **自建 FSM + scheduler** | 在 pi-agent-core 上自建（extension 形態） | 主路線（見下） |

## 建議架構（草稿）

**自建 workflow engine 作為 pi extension**，分四件：

1. **WorkflowJSON schema + 驗證器**（抄 n8n 的 schema 設計；LLM 生成後先驗再動）
2. **執行引擎**：FSM；節點執行器三種——simple llm（pi-ai 單 call）/ mini agent（pi-subagents 起 child）/ code（sandbox）
3. **Scheduler**：BullMQ（Redis）或 node-cron + 持久化 job registry；cron expression / 時區 / retry / 執行歷史
4. **MCP expose**：統一 Drill MCP server 動態註冊每條 pipeline 為一個 tool（input/output 型別從節點推導；跑在 [mcp/pi.md](../mcp/pi.md) 的 adapter 上）

可參考：pi-dynamic-workflows 的 journal resume 模式（long-running pipeline 的 checkpoint）。

## Scheduled Search template（tracer bullet）

對應 [spec.md](spec.md) 的第一個 built-in template。Track A 不另建 agent，直接組合既有搜尋工具與自建 scheduler：

| Template 元件 | Track A 對應 |
|---|---|
| arXiv source（預設） | `hinsencamp/pi-research-agent`（Semantic Scholar + arXiv + OpenAlex）或 `@wienerberliner/pi-arxiv`；全文轉換可接 `pi-arxivist` |
| 其他 Web sources | `pi-web-access` 的 `web_search` / `fetch_content` 與 provider fallback chain |
| cron / background run | 自建 BullMQ 或 node-cron + 持久化 job registry；不可依賴前端 session |
| normalize / dedupe | code node；優先鍵 arXiv ID → DOI → canonical URL |
| persistence | workspace collection / 論文庫 artifact；保存 provenance 與 run ID |
| status | workflow run history + last/next run；供 system-status 與 template UI 讀取 |

Tracer bullet 先固定 `sources = [arxiv]`，但 WorkflowJSON 使用 source adapter array，後續加入其他 web search tools 不需改 schema。

## 自建 gap 清單

1. Workflow engine 本體（上述四件）
2. pipeline builder agent 的 tool schema（`pipeline_create` / `_connect` / `_set_cron` / `_expose_mcp` / `_run` / `_status`——spec 定義）
3. DAG editor 前端（拖放 + inspector + 執行記錄）
4. Scheduled Search template manifest、source adapter registry、normalize/dedupe node 與 arXiv e2e fixture
