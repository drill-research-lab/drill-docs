# Design: Research Mode

> 對應 manpage.md「research mode」：實驗 / PoC 模式，coding agent 風格。
> Agent: **researcher**

## 定位

- 建立**實驗或 PoC**
- 可以**編輯程式碼**
- LLM 生成的文件或程式碼可讓用戶**點開在右邊查看、框起來 suggest fix 或編輯**
- 也可以編輯任意程式碼
- 像 coding agent（claude / codex / opencode）
- 用起來像 **vscode + copilot**

## Agent 設計

### researcher

- 負責**實驗設計、程式碼實作、PoC 建置**
- 參考：OpenCode / Codex / Claude Code 的 coding agent 範式
- **核心工具**：
  - `read` / `write` / `edit` / `bash` / `grep` / `glob` — 檔案與執行
  - `run_experiment(script, params)` — 執行實驗
  - `sandbox_exec(command)` — 沙盒內執行（隔離）
  - `analyze_results(data)` — 分析實驗結果
- **可以被 call**：normal 的 orchestrator（「跑個 PoC 驗證這個想法」）

## 協作模式

manpage 提到「LLM 生成的東西讓用戶點開查看、框起來 suggest fix」——這是 **agent + 用戶共編**：

```
LLM 寫 code → 用戶在右側檢視 → 框選片段 → suggest fix → LLM 修改 → 用戶確認 → 執行/驗證
```

### 內建 tool schema（coding agent）

| Tool | 用途 |
|---|---|
| `read` / `write` / `edit` | 檔案操作（pi-coding-agent 內建）|
| `bash` / `grep` / `glob` / `ls` | 執行與搜尋 |
| `sandbox_exec` | 沙盒執行（isolate 環境）|
| `lsp_diagnostics` | 即時 type check（pi-lens 模式）|
| `test` | 跑測試 |
| `git_diff` / `git_commit` | 版本控制 |

## 沙盒（sandbox）

實驗/實作需要 code 執行環境。選項：

| 方案 | 隔離 | 啟動 | 備註 |
|---|---|---|---|
| **Docker per run** | 中等 | ~1-2s | 成熟；v1 建議 |
| **gVisor / Firecracker microVM** | 強 | 稍慢 | v2 |
| **Gondolin（PI extension）** | micro-VM | — | PI 生態現成 |
| **E2B / Daytona 雲沙盒** | 雲端 | 快 | 要聯網、有費 |
| **context-mode / pi-landstrip** | 多層 | — | PI 生態現成 |

- **網路白名單**：允許 paper API / PyPI / npm；阻擋其他
- **Quota**：CPU / memory / walltime；timeout 強制 kill
- **產出**：code / 圖 / log 寫回專案 workspace（MinIO 或本地）

## UI 需求

- **編輯器**：CodeMirror 6 或 Monaco（vscode-like），支援 diff view
- **suggest fix 流程**：框選 → inline diff → accept/reject
- **終端**：內嵌 terminal（跑命令、看輸出）
- **檔案樹**：左側專案檔案瀏覽
- **實驗儀表板**：實驗參數、狀態、結果比較（A/B）
- **沙盒指示器**：目前 code 在哪個環境跑

## 整合點

- **Wiki**：實驗產出的知識（結果、insight）回寫知識庫 → librarian
- **Pipeline**：實驗可以包成 pipeline 節點（code 節點）
- **Paper**：實驗結果 → writer 寫進 paper
- **Review**：實驗數據可送 reviewer 驗證

## 開放問題

1. **共編模式**：agent 直接寫檔案 + 用戶看，還是 agent 提案 + 用戶 accept？（manpage 看起來偏後者）
2. 沙盒 v1 就做還是 lab mode 延後？（之前討論傾向延後）
3. 實驗結果的 reproducibility：要記錄完整環境（container image / seed / params）嗎？
4. researcher 和 pipeline 的 code 節點關係？（同一執行引擎？）
