# Features — 功能區導覽

> Drill 的 feature specs。產品名稱以 [Product Vocabulary](../product-vocabulary.md) 為準；既有 subfolder 名稱作為 legacy internal paths 暫不搬移。每個既有功能維持三檔結構：
>
> | 檔案 | 角色 | 讀者 |
> |---|---|---|
> | **[spec.md](normal/spec.md)** | 共同契約（*what*）：定位、agent、UI 需求、整合點、開放問題——**軌道中立** | 所有人（含自進化 agent 的 spec 輸入） |
> | **pi.md** | Track A（PI-based）設計（*how*）：複用組合、實作選項、自建 gap | PI 線開發 |
> | **dsh.md** | Track B（DSH-based）註記（*how*）：seam/plugin 對應、preset 對應 | DSH 線開發（阿柏） |
>
> 軌道總覽：[tracks/pi.md](../tracks/pi.md) · [tracks/dsh.md](../tracks/dsh.md)。決策（雙軌策略等）：[decisions.md](../decisions.md)。

## Feature specs

| 產品名稱 | Legacy path | Agent | 定位 | 狀態 |
|---|---|---|---|---|
| [Chat](normal/spec.md) | `normal` | Orchestrator 等 selectable modes | Conversations 與 agent 互動入口 | spec ✅ / [pi](normal/pi.md) ✅ / [dsh](normal/dsh.md) ✅ |
| [Library](wiki/spec.md) | `wiki` | Librarian | 類似 NotebookLM 或 llm-wiki；管理 Sources、論文庫、Wiki pages 與可重用研究資料 | spec ✅ / [pi](wiki/pi.md) ✅ / [dsh](wiki/dsh.md) ✅ |
| [Pipelines](pipeline/spec.md) | `pipeline` | Pipeline Builder | n8n 風格 workflow + cron（節點 = code/agent/llm） | spec ✅ / [pi](pipeline/pi.md) ✅ / [dsh](pipeline/dsh.md) ✅ |
| [Lab](research/spec.md) | `research` | Researcher | 實驗 / PoC、coding agent + sandbox | spec ✅ / [pi](research/pi.md) ✅ / [dsh](research/dsh.md) ✅ |
| [Writing](paper/spec.md) | `paper` | Writer | Overleaf 風格 paper / report / proposal 協作；名稱暫定 | spec ✅ / [pi](paper/pi.md) ✅ / [dsh](paper/dsh.md) ✅ |
| [Review](review/spec.md) | `review` | Reviewer | 無污染上下文事實驗證、anti-hallucination | spec ✅ / [pi](review/pi.md) ✅ / [dsh](review/dsh.md) ✅ |
| [Skills](skills/spec.md) | `skills` | — | SKILL.md 管理（add / import / edit） | spec ✅ / [pi](skills/pi.md) ✅ / [dsh](skills/dsh.md) ✅ |
| [MCP Servers](mcp/spec.md) | `mcp` | — | MCP server 管理（add / import / edit；內部 agent 用） | spec ✅ / [pi](mcp/pi.md) ✅ / [dsh](mcp/dsh.md) ✅ |
| [Settings](setting/spec.md) | `setting` | — | LLM、sandbox、approval、Project 等設定 | spec ✅ / [pi](setting/pi.md) ✅ / [dsh](setting/dsh.md) ✅ |
| [System Status](system-status/spec.md) | `system-status` | — | backend / frontend / LLM / network / storage 監控 | spec ✅ / [pi](system-status/pi.md) ✅ / [dsh](system-status/dsh.md) ✅ |

## 跨功能頁面

| 產品名稱 | 定位 | Spec |
|---|---|---|
| Notebook | Conversation Notes 的集中瀏覽入口，不是第二套 Wiki | [conversation-note.md](../conversation-note.md) |
| Files | Workspace / Project 的檔案與產出物入口 | [Workspace / Project ownership](../workspace-project-ownership.md) |

## 候選功能（尚未納入 10 個正式功能）

| 功能 | Agent | 定位 | 狀態 |
|---|---|---|---|
| [translation](translation/spec.md) | — | file-in / translated-file-out；是否值得 built-in 待討論 | candidate spec ⚠️；尚無 pi/dsh 設計 |

## 功能間依賴（示意）

```
Chat（Orchestrator）──call──► Library / Lab / Writing / Review / Pipelines
Review（Reviewer）◄──sync 驗證── Writing / Lab / Chat / Library
Lab ──結果回寫──► Library（Librarian）/ Writing（Writer）
Pipelines ──cron/MCP 觸發──► 任何 agent 節點；expose 成 tool 給其他 agent
skills / mcp / setting ──設定面──► 所有 agent
system-status ──監控──► 全部
```

## 撰寫規則

1. **spec.md 軌道中立**：spec 放**設計決策**（what），實作相關（plugin/seam/工具選型）放 pi.md / dsh.md
2. **有 agent 的功能必須有 `## Agent 設計` 區塊**（慣例與 normal 對齊）：`### <agent 名>` + 職責、核心工具、可被誰 call；管理頁類功能（skills/mcp/setting/system-status）無 agent、無此區塊
3. **pi.md 是深化主戰場**：複用組合（引用 [reference/pi-ecosystem.md](../reference/pi-ecosystem.md)）→ gap 清單 → 建議路線
4. **dsh.md 先薄後厚**：seam/preset 對應先列，實作細節等 fork 進度
5. 檔案長大可升級成目錄（`normal/pi.md` → `normal/pi/`），上層結構不動
