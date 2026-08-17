# MCP 管理頁 — Track A（PI-based）設計

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/pi.md](../../tracks/pi.md)。
> 原則：最大化複用 PI 生態系（[reference/pi-ecosystem.md](../../reference/pi-ecosystem.md)），gap 才自建。

## 整合層：pi-mcp-adapter（自 spec 移入；DeepWiki 已驗證）

PI 的 **pi-mcp-adapter** 提供完整 MCP bridge——此頁的功能底座：
- **proxy tool**：單一 `mcp` tool，agent 按需 search/describe/call（省 context）
- **direct tools**：常用 MCP tools 提升為一等公民（`directTools` config）
- **3 種 transport**：stdio / StreamableHTTP（含 SSE fallback）
- **OAuth 2.1**：dynamic client registration + PKCE；token 存儲（`~/.pi/agent/mcp-oauth/`）
- **4 層 config**：user-global / pi-global / project / pi-project（`.mcp.json`）
- lazy connect（預設）+ idle timeout（10min）；output guard（大輸出截斷存檔，路徑給 agent）
- 安裝：`pi install npm:...`

**UI 對應**：此頁的 server 列表/狀態燈/direct-vs-proxy 設定 ≈ pi-mcp-adapter 的 config 面——多為包裝其 config 的前端。

## 自建 gap 清單

1. Web UI（adapter 是 CLI/config 形態，需要管理頁前端 + config API）
2. pipeline 產生的 MCP servers 之動態註冊（統一 server 路線——見 [pipeline/spec.md](../pipeline/spec.md) 開放問題 #2）
3. 權限設定面（哪些 agent 可用此 server）——adapter 沒有 per-agent 權限，需疊在 permission 層
