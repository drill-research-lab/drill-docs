# Pipeline Mode — Track B（DSH-based）註記

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
