# System Status 頁 — Track A（PI-based）設計

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/pi.md](../../tracks/pi.md)。
> 原則：最大化複用 PI 生態系（[reference/pi-ecosystem.md](../../reference/pi-ecosystem.md)），gap 才自建。

## 複用組合

| 需求 | 複用 |
|---|---|
| LLM/turn/tool tracing | **@braintrust/pi-extension**（session/turn/LLM/tool 自動 tracing 到 Braintrust；自建 dashboard 的資料源之一）|
| 事件 stream | pi session log（JsonlSessionRepo / sqlite backend）+ extension 事件（`pi.on()` 全事件面）|
| token 用量 | pi-ai 的 usage 回報（provider 層）|

## 自建 gap 清單

1. metrics 收集與聚合（自建輕量或 Prometheus）
2. WebSocket 即時推送層
3. dashboard 前端（總覽卡/時間序列/drill-down）
4. per-user workspace 用量（D3——storage 監控主語意）
