# Features — 功能區導覽

> Drill 的 10 個功能（= 前端側邊欄）。每個功能一個 subfolder，三檔結構：
>
> | 檔案 | 角色 | 讀者 |
> |---|---|---|
> | **[spec.md](normal/spec.md)** | 共同契約（*what*）：定位、agent、UI 需求、整合點、開放問題——**軌道中立** | 所有人（含自進化 agent 的 spec 輸入） |
> | **pi.md** | Track A（PI-based）設計（*how*）：複用組合、實作選項、自建 gap | PI 線開發 |
> | **dsh.md** | Track B（DSH-based）註記（*how*）：seam/plugin 對應、preset 對應 | DSH 線開發（阿柏） |
>
> 軌道總覽：[tracks/pi.md](../tracks/pi.md) · [tracks/dsh.md](../tracks/dsh.md)。決策（雙軌策略等）：[decisions.md](../decisions.md)。

## 功能列表

| 功能 | Agent | 定位 | 狀態 |
|---|---|---|---|
| [normal](normal/spec.md) | orchestrator | 主聊天介面，可 call 其他 agent 做任何事情 | spec ✅ / [pi](normal/pi.md) ✅ / [dsh](normal/dsh.md) ✅ |
| [wiki](wiki/spec.md) | librarian | NotebookLM 風格知識庫（上傳文件 → LLM 整理） | spec ✅ / [pi](wiki/pi.md) ✅ / [dsh](wiki/dsh.md) ✅ |
| [pipeline](pipeline/spec.md) | pipeline builder | n8n 風格 workflow + cron（節點 = code/agent/llm） | spec ✅ / [pi](pipeline/pi.md) ✅ / [dsh](pipeline/dsh.md) ✅ |
| [research](research/spec.md) | researcher | 實驗/PoC，coding agent + 沙盒 | spec ✅ / [pi](research/pi.md) ✅ / [dsh](research/dsh.md) ✅ |
| [paper](paper/spec.md) | writer | Overleaf 風格論文/報告撰寫（LaTeX/MD）；報告產生器＝模板應用（國科會只是其中一個模板） | spec ✅ / [pi](paper/pi.md) ✅ / [dsh](paper/dsh.md) ✅ |
| [review](review/spec.md) | reviewer | 無污染上下文事實驗證，anti-幻覺 | spec ✅ / [pi](review/pi.md) ✅ / [dsh](review/dsh.md) ✅ |
| [skills](skills/spec.md) | — | SKILL.md 管理（add / import / edit） | spec ✅ / [pi](skills/pi.md) ✅ / [dsh](skills/dsh.md) ✅ |
| [mcp](mcp/spec.md) | — | MCP server 管理（add / import / edit；內部 agent 用） | spec ✅ / [pi](mcp/pi.md) ✅ / [dsh](mcp/dsh.md) ✅ |
| [setting](setting/spec.md) | — | 設定（LLM/沙盒/審核/專案） | spec ✅ / [pi](setting/pi.md) ✅ / [dsh](setting/dsh.md) ✅ |
| [system-status](system-status/spec.md) | — | 監控儀表板（backend/frontend/llm/network/storage） | spec ✅ / [pi](system-status/pi.md) ✅ / [dsh](system-status/dsh.md) ✅ |

## 候選功能（尚未納入 10 個正式功能）

| 功能 | Agent | 定位 | 狀態 |
|---|---|---|---|
| [translation](translation/spec.md) | — | file-in / translated-file-out；是否值得 built-in 待討論 | candidate spec ⚠️；尚無 pi/dsh 設計 |

## 功能間依賴（示意）

```
normal（orchestrator）──call──► wiki / research / paper / review / pipeline
review（reviewer）◄──sync 驗證── paper / research / normal / wiki
research ──結果回寫──► wiki（librarian）/ paper（writer）
pipeline ──cron/MCP 觸發──► 任何 agent 節點；expose 成 tool 給其他 agent
skills / mcp / setting ──設定面──► 所有 agent
system-status ──監控──► 全部
```

## 撰寫規則

1. **spec.md 軌道中立**：spec 放**設計決策**（what），實作相關（plugin/seam/工具選型）放 pi.md / dsh.md
2. **有 agent 的功能必須有 `## Agent 設計` 區塊**（慣例與 normal 對齊）：`### <agent 名>` + 職責、核心工具、可被誰 call；管理頁類功能（skills/mcp/setting/system-status）無 agent、無此區塊
3. **pi.md 是深化主戰場**：複用組合（引用 [reference/pi-ecosystem.md](../reference/pi-ecosystem.md)）→ gap 清單 → 建議路線
4. **dsh.md 先薄後厚**：seam/preset 對應先列，實作細節等 fork 進度
5. 檔案長大可升級成目錄（`normal/pi.md` → `normal/pi/`），上層結構不動
