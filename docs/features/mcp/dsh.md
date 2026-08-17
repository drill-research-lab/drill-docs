# MCP 管理頁 — Track B（DSH-based）註記

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/dsh.md](../../tracks/dsh.md)。
> 深度調查：[reference/dsh.md](../../reference/dsh.md)。

## seam / plugin 對應

| spec 需求 | DSH 對應 |
|---|---|
| MCP client/bridge | `dsh-mcp-client`（plugin per server 註冊 tools；DSH 內建，預設不啟用——見 [reference/dsh.md](../../reference/dsh.md)）|
| 4 層 `.mcp.json` / OAuth | 需確認 fork 上的行為（待查）|
| 管理 UI | DSH web UI 有 `ui-settings*` / plugin-inventory 面——可在此之上加 MCP 管理頁 |
