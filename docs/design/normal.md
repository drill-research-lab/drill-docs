# Design: Normal Mode

> 對應 manpage.md「normal mode」：主聊天介面，orchestrator agent。

## 定位

- **Drill 的主入口**：使用者進入的預設頁面
- 像 ChatGPT/Gemini Web 的聊天體驗
- **核心角色 = orchestrator**：可以 call 其他 agent as subagent 做任何事情
- 例：call librarian 查知識庫內容、call reviewer 驗證事實、call researcher 跑實驗、call writer 寫論文段落

## Agent 設計

### 主要 agent：orchestrator（main）

- 參考：HERMES Agent 的範式（`docs/reference.md` §2）
- **行為約束**：知道所有其他 agent 的存在與能力，能自行判斷何時該委派哪個 subagent
- **核心工具**：
  - `spawn_subagent(agent_type, task, context_mode)` — 派遣任何 agent（sync / async）
  - `read` / `write` / `edit` / `bash` — 基礎工具
  - 其他 agent 註冊的 tools（librarian 的 `kb_search`、reviewer 的 `verify_claims`…）
- **知識來源**：AGENTS.md + 目前 project context + 各 agent 的能力描述

### 可呼叫的 subagents（全部 peer）

| Agent | 用途 | 來源 mode |
|---|---|---|
| librarian | 知識庫查詢 / 整理 | wiki mode |
| researcher | 實驗 / PoC / 程式碼 | research mode |
| writer | 論文撰寫 / LaTeX | paper mode |
| reviewer | 事實驗證 / 審查 | review mode |
| pipeline builder | 建立 pipeline | pipeline mode |
| mini agent / simple llm | 輕量單次任務 | pipeline mode |

### 參考：DSH 四個 agent presets（default agent 分層的現成模型）

> DSH 預設四個 agent preset（`standard` / `minimal` / `code` / `cordis`，定義在 `agent.cordis.yml`；詳細見 [DSH.md](../DSH.md) §9）。對 Drill 的 agent 分層（主力 agent ↔ pipeline 節點 agent ↔ 自進化 meta-agent）是直接可抄的參考：

| DSH preset | 內容（DeepWiki 實證 2026-08-16） | Drill 對應 |
|---|---|---|
| `standard` | 完整 agent：persona + instructions + 全 toolset（e2e 實測 23 個 tools，含 `ralph`、`workflow`、`subagent`/`subagent_fork`、`send_message`、`interrupt_agent`…） | orchestrator / researcher / writer 等主力 agents |
| `minimal` | 固定 prompt 雙 tool agent（僅 persistent bash + str_replace_editor），`complete: true` + 無 runtime context、無 compaction | pipeline 的 **simple llm / mini agent 節點**（輕量、可預測） |
| `code` | standard 全部 + `tool-presentation` row → Code Mode（model 寫 TS 打 generated SDK，五個 round trip 變一個） | researcher 的程式密集任務（可選強化） |
| `cordis` | standard 全部 + self-referential toolset（`cordis_mount` 直接對 live runtime 跑 model 寫的 JS；官方定位「**建 custom agent presets** + runtime inspection + plugin 實驗」）；TRUST 標示「等同 shell access」 | **D2 自進化 meta-agent**（讀 spec → 生成/調整 plugin → 沉澱成 preset） |

Preset 機制本身也值得抄（DeepWiki 實證）：preset = 目錄 + `agent.cordis.yml`；roster 每 process mount 一次（standing scope），各 session 以 scope parentage 加入；service row 必須在 `isolate` realm 內否則 mount 時直接 reject（防跨 session 撞名）；authoring 只允許 copy（`ctx.agentPresets.copy`，唯一寫入），id 限制 `[a-z0-9][a-z0-9-]*`。

## Subagent 派遣機制（核心 primitive）

> 參考：OpenCode task tool + oh-my-openagent + pi-subagents（`docs/reference.md` §1, §9）

### 需求

1. **任意 agent 可 call 任意 agent**（peer-to-peer，非嚴格 hierarchy）
2. **sync + async 兩種模式**
   - sync：parent 等待 child 完成，拿最終結果（適用 reviewer 驗證）
   - async：parent 繼續，child 完成後通知（適用長 pipeline）
3. **context 隔離模式**（至少 3 種，要列為 enum）：
   - `fully-isolated`：reviewer 用，零污染
   - `shared-project`：librarian/wiki 用，繼承 project 知識
   - `fresh-inherits-task`：pipeline node 用，只有 task 描述
4. **深度 / 遞迴保護**：`max_depth`（pi-subagents 的 `maxSubagentDepth` 預設 2；OpenCode `subagent_depth` 預設 1——DeepWiki 實證）
5. **權限繼承**：child 繼承 parent 的 deny rules；預設禁止 `spawn_subagent` 無限遞迴

### 規格草案

```typescript
type ContextMode = "fully-isolated" | "shared-project" | "fresh-inherits-task";

interface SpawnSubagentParams {
  agentType: string;                    // librarian | researcher | writer | reviewer | ...
  task: string;                         // 要 child 做的任務描述
  contextMode?: ContextMode;            // 預設 "fresh-inherits-task"
  sync?: boolean;                       // 預設 true
  maxTokens?: number;                   // child 輸出上限
  timeoutMs?: number;                   // 預設 5min
}
```

### 實作選項

| 選項 | 說明 | 取捨 |
|---|---|---|
| **pi-subagents**（現成） | `subagent()` tool + workflowScript + builtin agents（scout/researcher/worker/reviewer/oracle） | 最快；但 builtin agents 是 coding 導向，需自訂 Drill agents |
| **oh-my-openagent task tool**（參考） | category-based routing + omo.jsonc agent 定義 | 架構最合；但它是獨立 harness 不是 library |
| **DSH subagents seam**（參考） | providers：spawn-in-process / fork / acp / codex / claude-code | 最完整；但要整包用 Cordis |
| **自建** | 在 pi-agent-core 上做 `spawn_subagent` tool | 完全掌控 context 隔離；工作量大 |

## UI 需求

- 聊天介面（markdown rendering + streaming）
- **subagent 活動指示器**：目前哪個 subagent 在跑、進度
- **審核節點**（若啟用）：agent 要 call 敏感 subagent 前顯示確認
- **上下文管理**：長對話的 compaction 提示、可切換 project
- **右側面板**（可選）：目前 session 的 agent 呼叫圖 / 執行樹

## 整合點

- **Auth**：`setting page` 管理
- **Session**：每 project 一個 session，可 resume
- **Memory**：⚠️ 未定——是否要記憶、是否提供 memory 開關設定、scope（per workspace/project vs per user）都還在思考。方向上欣賞 HERMES 的 **skill 機制**（程序性知識的自演化，見 [skills.md](skills.md) 參考模型），而非完整 HERMES memory 範式
- **Skills**：`skills` 頁管理，normal mode 可用 `/skill:name`

## 開放問題

1. **審核節點**要預設開啟還是關閉？（全自主 vs assistive 的平衡）
2. **normal mode 的記憶設計（未定）**：是否需要記憶還在思考；可能提供 memory 開關（setting）；scope 待定（per workspace/project vs per user）。目前傾向：程序性知識走 HERMES skill 機制（見 [skills.md](skills.md)），長期對話記憶另案討論
3. subagent 結果要不要全量進 parent context，還是只進 summary？
