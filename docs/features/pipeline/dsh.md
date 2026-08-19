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
