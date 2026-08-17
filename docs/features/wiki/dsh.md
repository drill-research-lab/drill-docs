# Wiki Mode — Track B（DSH-based）註記

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/dsh.md](../../tracks/dsh.md)。
> 深度調查：[reference/dsh.md](../../reference/dsh.md)。

## seam / plugin 對應

| spec 需求 | DSH 對應 |
|---|---|
| 文件儲存 | `ctx.fs` seam（`fs-local` provider，可換 sandboxed/workspace 化的 provider）|
| 檢索/索引 | 無內建 RAG primitive——需自建 plugin（或經 MCP 接 pi-knowledge 等外部引擎）|
| 文件解析 | markitdown/anydoc 為獨立工具（語言中立），以子進程或 MCP 接入 |
| KB MCP | 經 `dsh-mcp-client` 消費外部 KB MCP server |
