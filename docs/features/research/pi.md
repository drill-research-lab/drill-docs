# Research Mode — Track A（PI-based）設計

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/pi.md](../../tracks/pi.md)。
> 原則：最大化複用 PI 生態系（[reference/pi-ecosystem.md](../../reference/pi-ecosystem.md)），gap 才自建。

## 沙盒選型（自 spec 移入）

| 方案 | 隔離 | 啟動 | 備註 |
|---|---|---|---|
| **Docker per run** | 中等 | ~1-2s | 成熟；起手建議 |
| **Gondolin**（PI in-tree extension） | micro-VM | — | PI 生態現成（`packages/coding-agent/examples/extensions/`）|
| **pi-landstrip** | 多層 | — | PI 生態現成（sandboxed Bash + process-backed agents）|
| **gVisor / Firecracker microVM** | 強 | 稍慢 | 後續強化 |
| **E2B / Daytona 雲沙盒** | 雲端 | 快 | 要聯網、有費 |

## 工具底座

- coding agent 核心：PI 的 coding agent（`pi-coding-agent`）內建 read/write/edit/bash——researcher 的工具面大多現成
- `lsp_diagnostics`：pi 生態的 LSP 整合（pi-lens 模式）
- `sandbox_exec`：包裝沙盒選型（上表）為 tool

## 自建 gap 清單

1. `run_experiment` / `analyze_results` tools（實驗域語意——PI 無對應，自建 extension）
2. 回溯機制（spec：agent 直接寫 + 可 rollback）——參考 PI in-tree `git-checkpoint` extension
3. 編輯器前端（CodeMirror 6 / Monaco、diff view、suggest-fix 流程、內嵌 terminal）
