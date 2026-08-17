# Review Mode — Track B（DSH-based）註記

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/dsh.md](../../tracks/dsh.md)。
> 深度調查：[reference/dsh.md](../../reference/dsh.md)。

## seam / plugin 對應

| spec 需求 | DSH 對應 |
|---|---|
| clean-context session | `subagent-spawn-in-process` provider（fresh child）；`SessionHeader.delegationDepth` 防遞迴 |
| 事實查核工具面 | 學術/搜尋工具經 MCP 或子進程接入（軌道無關，見 [paper/dsh.md](../paper/dsh.md) 同模式）|
| model override（spec：reviewer 可設定） | per-preset `agent-default-model`——reviewer preset 可指向不同 provider |
| 送審人類節點 | approval policy（dsh-base 內建）|
