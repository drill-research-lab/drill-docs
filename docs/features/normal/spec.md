# Design: Chat

> Legacy internal path：`normal`。產品名稱 **Chat**，可選擇不同 Agent / Mode。

## 定位

- **Drill 的主入口**：使用者進入的預設頁面
- 像 ChatGPT/Gemini Web 的聊天體驗
- **Orchestrator mode** 可以 call 其他 agent as subagent 做任何事情
- 例：call librarian 查知識庫內容、call reviewer 驗證事實、call researcher 跑實驗、call writer 寫論文段落

## Agent 設計

### Agent / Mode selector

Chat 提供三個平級選項；完整命名基線見 [product-vocabulary.md](../../product-vocabulary.md)：

| 名稱 | 定位 | 能力輪廓 |
|---|---|---|
| **Standard** | 預設完整工作模式 | 檔案編輯、Shell、檔案與 Web 檢索、Skills、計畫、目標、subagents、workflows |
| **Orchestrator** | 更強的協調與自主執行角色 | 以 Standard 為基礎，具有更高權限與更積極的多 agent 協調；精確邊界待定 |
| **Minimal** | 刻意縮小工具面的工作模式 | 與 Standard / Orchestrator 平級，不列為 Advanced；PI / DSH 的確切工具由軌道檔定義 |

`Temporary Chat` 是 Conversation 的保存／記憶選項，不是第四個 Agent / Mode。Minimal 對部分模型／任務可能更穩定或有效率，但不宣稱普遍優於 Standard。

### OpenCode / oh-my-openagent 設計參考

| 生態 | 參考 agent | 可借設計 | 匹配邊界 |
|---|---|---|---|
| OpenCode | `build`（primary） | 預設主互動 agent、廣泛工具權限；以 Task tool 派遣 subagent，並用 `permission.task` 控制可呼叫名單 | 最接近入口與工具面，但不是專門命名的 orchestrator |
| oh-my-openagent | `Sisyphus`（primary） | 規劃、category / `subagent_type` 雙路派工、平行執行、驗收並驅動任務完成 | **直接對應** Drill orchestrator |

DeepWiki 查證位置：OpenCode `packages/opencode/src/agent/agent.ts`、`packages/opencode/src/tool/task.ts`；omo `docs/guide/orchestration.md`。完整 roster → [reference/domains.md](../../reference/domains.md) §1。

### Orchestrator mode agent

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
| librarian | 知識庫查詢 / 整理 | Library |
| researcher | 實驗 / PoC / 程式碼 | Lab |
| writer | 論文撰寫 / LaTeX | Writing |
| reviewer | 事實驗證 / 審查 | Review |
| pipeline builder | 建立 pipeline | Pipelines |
| mini agent / simple llm | 輕量單次任務 | Pipelines |

### 參考：DSH 四個 agent presets（default agent 分層的現成模型）

> DSH 的 preset 分層（Standard / Code / Minimal / Creator，內部 ID `standard` / `code` / `minimal` / `cordis`）是直接參考；Minimal 同時可作使用者選項與 pipeline 節點。完整對照表見 [dsh.md](dsh.md)（Track B 註記）與 [reference/dsh.md](../../reference/dsh.md) §9。

## Subagent 派遣機制（核心 primitive）

> 參考：OpenCode task tool + oh-my-openagent + pi-subagents（[reference/domains.md](../../reference/domains.md) §1、[reference/pi-ecosystem.md](../../reference/pi-ecosystem.md)）

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

各軌道的實作選項：**Track A**（pi-subagents / omo 參考 / 自建於 pi-agent-core）→ [pi.md](pi.md)；**Track B**（DSH subagents seam）→ [dsh.md](dsh.md)。

## UI 需求

- 聊天介面（markdown rendering + streaming）
- **PDF 產出物**：Agent 產生或修改 PDF 後，對應訊息直接提供「預覽」與「下載」；預覽留在 Drill 內，沿用 Writer / Wiki 共用的 PDF viewer
- **Conversation Note**：每個 Conversation 可產生／更新一份 Markdown Note；可請 LLM 修正、直接編輯或重新產生；完整規格見 [conversation-note.md](../../conversation-note.md)
- **Notebook**：集中瀏覽 Workspace / Project 內的 Conversation Notes，不作為第二套 Wiki
- **subagent 活動指示器**：目前哪個 subagent 在跑、進度
- **審核節點**（若啟用）：agent 要 call 敏感 subagent 前顯示確認
- **上下文管理**：長對話的 compaction 提示、可切換 project
- **右側面板**（可選）：目前 session 的 agent 呼叫圖 / 執行樹

## 整合點

- **Auth**：`setting page` 管理
- **Session**：每 project 一個 session，可 resume
- **Memory**：⚠️ 未定——是否要記憶、是否提供 memory 開關設定、scope（per workspace/project vs per user）都還在思考。方向上欣賞 HERMES 的 **skill 機制**（程序性知識的自演化，見 [skills/spec.md](../skills/spec.md) 參考模型），而非完整 HERMES memory 範式
- **Skills**：Skills 頁管理，Chat 可用 `/skill:name`

## 開放問題

1. **審核節點**要預設開啟還是關閉？（全自主 vs assistive 的平衡）
2. **Chat 的記憶設計（未定）**：是否需要記憶還在思考；可能提供 memory 開關（Settings）；scope 待定（per Workspace / Project vs per user）。目前傾向：程序性知識走 HERMES skill 機制（見 [skills/spec.md](../skills/spec.md)），長期對話記憶另案討論
3. subagent 結果要不要全量進 parent context，還是只進 summary？
