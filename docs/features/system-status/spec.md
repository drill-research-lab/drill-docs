# Design: System Status 頁

> 對應 index.md（原 manpage）「system status」：system status / backend / frontend / llm / network / storage..etc

## 定位

- 系統狀態總覽：**backend / frontend / llm / network / storage** 等
- 維運監控儀表板

## 監控維度

### 1. Backend
- 服務健康：API server、workflow engine、scheduler、memory service
- 各服務的：uptime、CPU、memory、錯誤率
- 活躍 agent sessions 數
- 隊列深度（pending tasks）

### 2. LLM
- **model 健康**：spark `:8000/v1` 心跳、回應延遲、token throughput
- 目前載入的 model（91 GiB DeepSeek-V4-Flash @ DGX Spark）
- **GPU / 記憶體使用率**（128 GB unified memory 的使用）
- **成本儀表板**（外部 API）：per-user / per-project token 消耗、費用
- model switching 狀態（載入 / 卸載進度）

### 3. Frontend
- WebSocket 連線數（active clients）
- 前端錯誤率
- 版本資訊

### 4. Network
- 外部 API 連線狀態（OpenAI / Anthropic / Google / Exa / Perplexity…）
- 延遲、rate limit 狀態
- 網路白名單狀態（sandbox 的 egress 控制）

### 5. Storage（D3：user workspace + 用量監控）
- **per-user workspace 用量**（D3 定義的 storage 主語意）：每個 user 的 workspace 目錄、配額使用、超額警告
- 基礎設施層用量：
  - Postgres（metadata / sessions / projects）
  - 向量 DB（embeddings）
  - Blob（papers / PDFs / artifacts）
  - Redis（hot state）
- 磁碟空間、備份狀態

### 6. Agents / Pipelines
- 活躍 agent 列表（哪個 mode、哪個 subagent、狀態）
- pipeline 執行歷史（成功率、平均時長、收斂率——每 stage loop 幾次）
- 排程任務（cron jobs 下次執行）

### 7. Sandbox
- sandbox pool 使用率
- 執行中容器數
- 配額使用（CPU / memory / walltime）

## 實作選項

| 方案 | 說明 |
|---|---|
| **自建 dashboard**（前端頁面 + WebSocket 即時更新）| 與 Drill UI 整合；推薦 |
| **Grafana + Prometheus** | 標準監控棧；但要多個服務 |

> 各軌道的 tracing/事件源補充 → [pi.md](pi.md) / [dsh.md](dsh.md)。

## 資料來源

- 各服務 health endpoint（`/health`）
- 事件 stream（append-only 事件 log——各軌道的 session log）
- 指標收集（自建輕量 metrics 或 Prometheus）
- **審計 log**（誰做了什麼——per-interaction attribution）

## UI 需求

- 總覽卡（各維度關鍵數字）
- 歷史圖表（時間序列）
- **即時更新**（WebSocket 推送，非 polling）
- 錯誤 / 警告列表（可點擊看細節）
- drill-down：點 LLM → 看具體 model / token 細節

## 開放問題

1. 監控要自建輕量版（v1）還是直接上 Grafana（v2）？
2. **成本追蹤**重要嗎？（外部 API 用量的可視化）
3. 審計 log（agent 行為追蹤）要做到多細？（Cordis 的 effect attribution 是全記錄 vs 只記錄重要事件）
