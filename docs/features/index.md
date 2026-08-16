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
| [normal](normal/spec.md) | orchestrator | 主聊天介面，可 call 其他 agent 做任何事情 | spec ✅ / [pi](normal/pi.md) 初稿 / [dsh](normal/dsh.md) 初稿 |
| [wiki](wiki/spec.md) | librarian | NotebookLM 風格知識庫（上傳文件 → LLM 整理） | spec ✅ / [pi](wiki/pi.md) 骨架 / [dsh](wiki/dsh.md) 骨架 |
| [pipeline](pipeline/spec.md) | pipeline builder | n8n 風格 workflow + cron（節點 = code/agent/llm） | spec ✅ / [pi](pipeline/pi.md) 骨架 / [dsh](pipeline/dsh.md) 骨架 |
| [research](research/spec.md) | researcher | 實驗/PoC，coding agent + 沙盒 | spec ✅ / [pi](research/pi.md) 骨架 / [dsh](research/dsh.md) 骨架 |
| [paper](paper/spec.md) | writer | Overleaf 風格論文撰寫（LaTeX/MD）＋ 🥇 第一個垂直切片（國科會報告產生器） | spec ✅ / [pi](paper/pi.md) 骨架 / [dsh](paper/dsh.md) 骨架 |
| [review](review/spec.md) | reviewer | 無污染上下文事實驗證，anti-幻覺 | spec ✅ / [pi](review/pi.md) 骨架 / [dsh](review/dsh.md) 骨架 |
| [skills](skills/spec.md) | — | SKILL.md 管理（add / import / edit） | spec ✅ / [pi](skills/pi.md) 骨架 / [dsh](skills/dsh.md) 骨架 |
| [mcp](mcp/spec.md) | — | MCP server 管理（add / import / edit；內部 agent 用） | spec ✅ / [pi](mcp/pi.md) 骨架 / [dsh](mcp/dsh.md) 骨架 |
| [setting](setting/spec.md) | — | 設定（LLM/沙盒/審核/專案） | spec ✅ / [pi](setting/pi.md) 骨架 / [dsh](setting/dsh.md) 骨架 |
| [system-status](system-status/spec.md) | — | 監控儀表板（backend/frontend/llm/network/storage） | spec ✅ / [pi](system-status/pi.md) 骨架 / [dsh](system-status/dsh.md) 骨架 |

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

1. **spec.md 軌道中立**：不提特定 plugin/seam 名；實作選項表放 pi.md / dsh.md
2. **pi.md 是深化主戰場**：複用組合（引用 [reference/pi-ecosystem.md](../reference/pi-ecosystem.md)）→ gap 清單 → 建議路線
3. **dsh.md 先薄後厚**：seam/preset 對應先列，實作細節等 fork 進度
4. 檔案長大可升級成目錄（`normal/pi.md` → `normal/pi/`），上層結構不動
