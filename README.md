# Drill 🌀 — 全自動研究系統設計文件

> **Drill（鑽頭）**：打造一個全自動研究平台：workflow pipeline 可自建、可被 cron 排程或 MCP toolcall 觸發。「知識來源 → 選題 → 搜尋 → 調查 → 實驗 → 報告，每階段可 loop」只是其中一種 pipeline 模板。
> 本 repo 存放 Drill 的**設計文件**（無程式碼）。

---

## 文件導覽

| 區 | 文件 | 內容 |
|---|---|---|
| 總覽 | [docs/index.md](docs/index.md) | **起點**。10 個功能的簡介與 spec 連結（側邊欄對應） |
| 總覽 | [docs/concepts.md](docs/concepts.md) | Agent 工程術語階梯：Prompt → Context → Harness → Loop → Graph engineering |
| 總覽 | [docs/decisions.md](docs/decisions.md) | 跨軌道決策紀錄（雙軌、自進化、開源政策…） |
| 通用參考 | [docs/reference/](docs/reference/) | 外部專案調查（與軌道無關）：[總覽＋命名陷阱](docs/reference/README.md) · [DSH 深度調查](docs/reference/dsh.md) · [Cordis paper 分析](docs/reference/cordis.md) · [PI 生態系 plugins](docs/reference/pi-ecosystem.md) · [分域調查](docs/reference/domains.md) |
| 雙軌 | [docs/tracks/pi.md](docs/tracks/pi.md) | **Track A（PI-based）**：複用地圖、自建 gap、組合架構 |
| 雙軌 | [docs/tracks/dsh.md](docs/tracks/dsh.md) | **Track B（DSH-based，團隊主線）**：fork 狀態、seam↔功能對應 |
| 功能 | [docs/features/](docs/features/) | 每個功能一個 subfolder：`spec.md`（共同契約）+ `pi.md`（Track A 設計）+ `dsh.md`（Track B 註記） |
| 歷史 | [docs/archive/disscuss.md](docs/archive/disscuss.md) | ⚠️ 已棄用。早期 Spark 階段討論（蒸餾策略、13-stage 隨手草稿） |

## 核心架構

**10 個功能**（左側邊欄），每個對應 `docs/features/<name>/`：

| 功能 | Agent | 定位 |
|---|---|---|
| [normal](docs/features/normal/spec.md) | orchestrator | 主聊天介面，可 call 其他 agent 做任何事情 |
| [wiki](docs/features/wiki/spec.md) | librarian | NotebookLM 風格知識庫（上傳文件 → LLM 整理） |
| [pipeline](docs/features/pipeline/spec.md) | pipeline builder | n8n 風格 workflow + cron（節點 = code/agent/llm） |
| [research](docs/features/research/spec.md) | researcher | 實驗/PoC，coding agent + 沙盒 |
| [paper](docs/features/paper/spec.md) | writer | Overleaf 風格論文撰寫（LaTeX/MD） |
| [review](docs/features/review/spec.md) | reviewer | 無污染上下文事實驗證，anti-幻覺 |
| [skills](docs/features/skills/spec.md) | — | SKILL.md 管理 |
| [mcp](docs/features/mcp/spec.md) | — | MCP server 管理 |
| [setting](docs/features/setting/spec.md) | — | 設定（LLM/沙盒/審核/專案） |
| [system-status](docs/features/system-status/spec.md) | — | 監控儀表板 |

**關鍵設計原則**：
- **任意 agent 可 call 任意 agent**（peer-to-peer subagent 派遣，sync/async）
- **研究法蒸餾**：調查大家的研究用法與痛點 → 抽象共同模式 → 沉澱成 workflow 模板（archive/disscuss.md 的 13-stage 表是早期討論隨手記下的草稿，非 spec）
- **複用優先**：PI 生態系 30+ plugins、DSH/Cordis、學術工具鏈
- **硬體約束**：DeepSeek-V4-Flash-0731 @ DGX Spark（128GB unified memory）

## 參考專案速覽

調查了 40+ 開源專案，重點：**OpenCode** / **oh-my-openagent** / **PI (pi.dev)** / **DeepSeek Harness** / **Cordis** / **HERMES** / **Honcho** / **n8n** / **Overleaf** — 詳細見 [docs/reference/](docs/reference/)。

## 狀態

- 📝 設計階段（2026-08-14 起）
- 🔍 持續調查參考專案（deepwiki / gh 實證）
- 🏗️ 後端 / 前端目錄為空，待設計定稿後開發

## 相關

- Org: [github.com/drill-research-lab](https://github.com/drill-research-lab)
- Fork 參考: [deepseek-harness](https://github.com/drill-research-lab/deepseek-harness)
