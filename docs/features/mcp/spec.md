# Design: MCP 管理頁

> 對應 index.md（原 manpage）「mcp」：add / import / edit mcps。

## 定位

- 管理 Drill 的 **MCP servers**（Model Context Protocol）
- 操作：**add / import / edit**
- MCP = Drill 的**通用工具協議**（所有外部 tools 透過 MCP 接入）

## MCP 是什麼（回顧）

- 開放協議：AI app ↔ external tools/data
- JSON-RPC 2.0；stdio / Streamable HTTP / SSE transports
- Server capabilities：tools / resources / prompts；Client：sampling / elicitation / logging
- 官方 repo 見 [reference/domains.md](../../reference/domains.md) §8

## 整合層

MCP client/bridge 的實作：**Track A** → [pi.md](pi.md)（pi-mcp-adapter）；**Track B** → [dsh.md](dsh.md)（dsh-mcp-client）。

## 頁面功能

### 1. 瀏覽 / 列表
- 所有已配置 MCP servers（名稱、transport、狀態：connected/disconnected/error）
- 每個 server 的 tools 列表（可 search / describe）
- **drill 特有**：pipeline 產生的 MCP servers（agent 建好的 pipeline 會 expose 成 MCP tool）

### 2. Add（新增）
- **stdio server**：`command + args`（本地 process，如 `npx @modelcontextprotocol/server-filesystem`）
- **HTTP server**：`url + headers`（遠端）
- 從 **npm package** 安裝（各軌道 installer——Track A 見 [pi.md](pi.md)）
- **認證**：OAuth 2.1 / bearer token（設 token 或 env var）

### 3. Import（匯入）
- `.mcp.json` 匯入（Claude Code / Cursor / OpenCode 生態的 config 格式相容）
- 從**其他 agent 生態**（Claude Code / Codex）搬移

### 4. Edit（編輯）
- 啟用 / 停用
- 認證更新
- **expose/direct tools 設定**：哪個 tool 提升為 direct（常駐）vs proxy（按需）
- 權限：哪些 agent 可用此 server 的 tools

## Drill 特有的 MCP 使用場景

| 場景 | 說明 |
|---|---|
| **研究工具** | 學術 DB / Zotero 類 MCP servers（例：paper-pilot）|
| **Web 研究** | web search / fetch 類工具（例：pi-web-access——Track A；或對應 MCP servers）|
| **Pipeline expose** | pipeline 建立後自動成為 MCP server，**內部** agents 可呼叫 |
| **知識庫** | KB 檢索 MCP（向量檢索）|
| ~~Drill 自身 expose~~ | 不做（見開放問題 #1）；Drill 的 MCP 皆為內部使用 |

## UI 需求

- server 列表 + 狀態燈
- 新增精靈（transport 類型 → 參數 → 測試連線）
- tools 檢視（每個 tool 的 schema + 試用）
- 認證管理（token 存儲、OAuth 狀態）
- 日誌（MCP 呼叫記錄、錯誤）

## 整合點

- **Skills**：skill frontmatter 可宣告需要的 MCP servers（skill-embedded MCP）
- **Pipeline**：pipeline 的 MCP expose 由此頁管理
- **Agent**：所有 agent 透過 MCP tool plane 接外部能力
- **Auth**：OAuth token 存儲與 credential store 整合

## 開放問題

1. ✅ 已解決（2026-08-16）：**不做 Drill 反向 expose**。Drill 的 MCP 是內部基礎設施（builder / config / setting，給內部 agents 消費，含 pipeline expose 的 tools）。若未來真要讓外部工具（IDE 等）接 Drill 的 agents，會選 ACP（DSH 內建）而非 MCP——但目前沒有這個計畫
2. MCP server 的**隔離**：第三方 MCP server 的程式碼（stdio process）要在沙盒跑嗎？
3. 每個 project 一份 MCP config 還是 global 一份？
