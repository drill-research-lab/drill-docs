# Design: Lab

> Legacy internal path：`research`。Lab 是實驗 / PoC 與 coding-agent 工作環境。
> Agent: **researcher**

## 定位

- 建立**實驗或 PoC**
- 可以**編輯程式碼**
- LLM 生成的文件或程式碼可讓用戶**點開在右邊查看、框起來 suggest fix 或編輯**
- 也可以編輯任意程式碼
- 像 coding agent（claude / codex / opencode）
- 用起來像 **vscode + copilot**

## Agent 設計

### OpenCode / oh-my-openagent 設計參考

| 生態 | 參考 agent | 可借設計 | 匹配邊界 |
|---|---|---|---|
| OpenCode | `build`（primary）+ `general`（subagent） | `build` 提供完整 editing / shell 工具的 coding-agent loop；`general` 承接 delegated multi-step research | **直接對應**程式實作與委派研究；實驗記錄、sandbox quota 仍是 Drill 擴充 |
| oh-my-openagent | `Hephaestus`（primary） | autonomous deep worker：codebase 探索、模式研究、跨檔實作與驗證，適合長時間複雜技術任務 | **直接對應**自主 researcher/coding agent；學術實驗工具不是其內建能力 |

DeepWiki 查證位置：OpenCode `packages/opencode/src/agent/agent.ts`；omo Hephaestus agent 定義與 agent-model matching 文件。完整 roster → [reference/domains.md](../../reference/domains.md) §1。

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

index.md（原 manpage）提到「LLM 生成的東西讓用戶點開查看、框起來 suggest fix」——這是 **agent + 用戶共編**：

```
LLM 寫 code → 用戶在右側檢視 → 框選片段 → suggest fix → LLM 修改 → 用戶確認 → 執行/驗證
```

### 內建 tool schema（coding agent）

| Tool | 用途 |
|---|---|
| `read` / `write` / `edit` | 檔案操作 |
| `bash` / `grep` / `glob` / `ls` | 執行與搜尋 |
| `sandbox_exec` | 沙盒執行（isolate 環境）|
| `lsp_diagnostics` | 即時 type check |
| `test` | 跑測試 |
| `git_diff` / `git_commit` | 版本控制 |

## 沙盒（sandbox）

實驗/實作需要 code 執行環境。隔離需求（安全邊界，設計決策）：

- **網路白名單**：允許 paper API / PyPI / npm；阻擋其他
- **Quota**：CPU / memory / walltime；timeout 強制 kill
- **產出**：code / 圖 / log 寫回專案 workspace

> 隔離技術選型（Docker / microVM / 雲沙盒——各軌道候選）→ [pi.md](pi.md) / [dsh.md](dsh.md)。

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

1. **共編模式（傾向，未定案）**：agent **直接寫入** + **回溯機制**（像 OpenCode 的 checkpoint/undo——寫了可回滾），用戶右側查看 + 框選 suggest fix
2. 沙盒 v1 就做還是 lab mode 延後？（之前討論傾向延後）
3. 實驗結果的 reproducibility：要記錄完整環境（container image / seed / params）嗎？
4. researcher 和 pipeline 的 code 節點關係？（同一執行引擎？）
