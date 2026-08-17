# System Status 頁 — Track B（DSH-based）註記

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/dsh.md](../../tracks/dsh.md)。
> 深度調查：[reference/dsh.md](../../reference/dsh.md)。

## seam / plugin 對應

| spec 需求 | DSH 對應 |
|---|---|
| 事件 stream | append-only `SessionEvent` log + `ctx.sessionQuery`（FTS5/lineage）|
| 審計 log | Cordis per-interaction attribution + telemetry（dsh-base 內建 telemetry plugin）|
| 執行軌跡 UI | `ui-trajectory`（按 source 檢視的現成面板）|
| 前端錯誤/連線 | web UI 的 `events.mux`/`events.host` WebSocket 層 |
