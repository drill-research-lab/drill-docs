# Drill 🌀 — 全自動研究系統設計文件

> **Drill（鑽頭）**：以 Workspace / Project 為核心的 AI research workstation，整合 Chat、Library、Pipelines、Lab、Writing、Review、Notebook 與 Files，讓研究資料、對話、實驗、流程與產出在同一工作環境中持續累積與重用。
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

產品名稱以 [Product Vocabulary](docs/product-vocabulary.md) 為準；既有 `docs/features/<name>/` 路徑暫時保留 legacy internal names：

| 產品名稱 | Legacy path | Agent | 定位 |
|---|---|---|---|
| [Chat](docs/features/normal/spec.md) | `normal` | Orchestrator 等 modes | Conversations 與 agent 互動入口 |
| [Library](docs/features/wiki/spec.md) | `wiki` | Librarian | 類似 NotebookLM 或 llm-wiki；管理 Sources、論文庫與 Wiki pages |
| [Pipelines](docs/features/pipeline/spec.md) | `pipeline` | Pipeline Builder | workflow、templates、cron 與 runs |
| [Lab](docs/features/research/spec.md) | `research` | Researcher | 實驗 / PoC、coding agent + sandbox |
| [Writing](docs/features/paper/spec.md) | `paper` | Writer | Overleaf 風格 paper / report / proposal 協作；名稱暫定 |
| [Review](docs/features/review/spec.md) | `review` | Reviewer | 無污染上下文事實驗證、anti-hallucination |
| [Skills](docs/features/skills/spec.md) | `skills` | — | SKILL.md 管理 |
| [MCP Servers](docs/features/mcp/spec.md) | `mcp` | — | MCP server 管理 |
| [Settings](docs/features/setting/spec.md) | `setting` | — | LLM、sandbox、approval、Project 等設定 |
| [System Status](docs/features/system-status/spec.md) | `system-status` | — | 監控儀表板 |

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
