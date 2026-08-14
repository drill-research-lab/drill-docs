# Design: MCP 管理頁

> 對應 manpage.md「mcp」：add / import / edit mcps。

## 定位

- 管理 Drill 的 **MCP servers**（Model Context Protocol）
- 操作：**add / import / edit**
- MCP = Drill 的**通用工具協議**（所有外部 tools 透過 MCP 接入）

## MCP 是什麼（回顧）

- 開放協議：AI app ↔ external tools/data
- JSON-RPC 2.0；stdio / Streamable HTTP / SSE transports
- Server capabilities：tools / resources / prompts；Client：sampling / elicitation / logging
- 官方 repo：`docs/reference.md` §8

## 整合層（複用 pi-mcp-adapter）

PI 的 **pi-mcp-adapter**（`docs/reference.md` §9）提供完整 MCP bridge：
- **proxy tool**：單一 `mcp` tool，agent 按需 search/describe/call（省 context）
- **direct tools**：常用 MCP tools 提升為一等公民
- **3 種 transport**：stdio / StreamableHTTP / SSE
- **OAuth 2.1**：dynamic client registration + token 存儲（`~/.pi/agent/mcp-oauth/`）
- **4 層 config**：user-global / pi-global / project / pi-project（`.mcp.json`）

## 頁面功能

### 1. 瀏覽 / 列表
- 所有已配置 MCP servers（名稱、transport、狀態：connected/disconnected/error）
- 每個 server 的 tools 列表（可 search / describe）
- **drill 特有**：pipeline 產生的 MCP servers（agent 建好的 pipeline 會 expose 成 MCP tool）

### 2. Add（新增）
- **stdio server**：`command + args`（本地 process，如 `npx @modelcontextprotocol/server-filesystem`）
- **HTTP server**：`url + headers`（遠端）
- 從 **npm package**：`pi install npm:...`（pi-mcp-adapter 生態）
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
| **研究工具** | paper-pilot MCP（6 學術 DB + Zotero）、Semantic Scholar、OpenAlex |
| **Web 研究** | pi-web-access（web_search / fetch_content / code_search）|
| **Pipeline expose** | pipeline 建立後自動成為 MCP server，其他 agent 可呼叫 |
| **知識庫** | KB MCP（向量檢索）|
| **Drill 自身 expose** | Drill 的 agent 可被外部 MCP client 呼叫（反向）|

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

1. Drill 自己要不要 expose 成 MCP server？（反過來讓外部工具呼叫 Drill 的 agents）——pi-mcp-adapter 目前只消費不提供，要自建
2. MCP server 的**隔離**：第三方 MCP server 的程式碼（stdio process）要在沙盒跑嗎？
3. 每個 project 一份 MCP config 還是 global 一份？
