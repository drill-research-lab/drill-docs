# Writing（名稱暫定）— Track B（DSH-based）註記

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/dsh.md](../../tracks/dsh.md)。
> 深度調查：[reference/dsh.md](../../reference/dsh.md)。

## seam / plugin 對應

| spec 需求 | DSH 對應 |
|---|---|
| writer agent | custom preset（copy `standard`，加上學術工具面）|
| LaTeX compile | 沙盒內 tectonic/latexmk（`ctx.fs` + sandbox）；無現成 CLSI 對應——自建 compile service plugin |
| 學術工具鏈 | pi 生態 plugins 多為 CLI/MCP 形態，可經 MCP（`dsh-mcp-client`）或子進程接入——軌道無關 |
| deliverables | web UI 有 `ui-deliverables` plugin（產出物面板，PDF 可掛）|
