# Drill 🌀 — 全自動研究系統設計文件

> **Drill（鑽頭）**：打造一個全自動研究平台，整合知識來源 → 選題 → 搜尋 → 調查 → 實驗 → 報告的完整 pipeline，每個階段皆可 loop 直到完善。
> 本 repo 存放 Drill 的**設計文件**（無程式碼）。

---

## 文件導覽

| 文件 | 內容 |
|---|---|
| [docs/manpage.md](docs/manpage.md) | **起點**。側邊欄總覽：10 個 mode 的簡介與設計文件連結 |
| [docs/disscuss.md](docs/disscuss.md) | 設計理念：13-stage research pipeline、蒸餾策略、參考系統 |
| [docs/reference.md](docs/reference.md) | 所有參考專案總覽（OSS/CLOSED/MAIN 標記、深度調查、命名陷阱校正） |
| [docs/DSH.md](docs/DSH.md) | DeepSeek Harness 深度設計調查（15 面向、commit 級驗證、Drill 決策輸入） |
| [docs/design/](docs/design/) | 每個 mode 的詳細設計（共 10 份） |

## 核心架構

**10 個 mode**（左側邊欄），每個對應一個 agent persona：

| Mode | Agent | 定位 |
|---|---|---|
| [normal](docs/design/normal.md) | orchestrator | 主聊天介面，可 call 其他 agent 做任何事情 |
| [wiki](docs/design/wiki.md) | librarian | NotebookLM 風格知識庫（上傳文件 → LLM 整理） |
| [pipeline](docs/design/pipeline.md) | pipeline builder | n8n 風格 workflow + cron（節點 = code/agent/llm） |
| [research](docs/design/research.md) | researcher | 實驗/PoC，coding agent + 沙盒 |
| [paper](docs/design/paper.md) | writer | Overleaf 風格論文撰寫（LaTeX/MD） |
| [review](docs/design/review.md) | reviewer | 無污染上下文事實驗證，anti-幻覺 |
| [skills](docs/design/skills.md) | — | SKILL.md 管理 |
| [mcp](docs/design/mcp.md) | — | MCP server 管理 |
| [setting](docs/design/setting.md) | — | 設定（LLM/沙盒/審核/專案） |
| [system-status](docs/design/system-status.md) | — | 監控儀表板 |

**關鍵設計原則**：
- **任意 agent 可 call 任意 agent**（peer-to-peer subagent 派遣，sync/async）
- **13-stage research pipeline** 每階段可 loop / 分支 / 回溯
- **複用優先**：PI 生態系 30+ plugins、DSH/Cordis、學術工具鏈
- **硬體約束**：DeepSeek-V4-Flash-0731 @ DGX Spark（128GB unified memory）

## 參考專案速覽

調查了 40+ 開源專案，重點：**OpenCode** / **oh-my-openagent** / **PI (pi.dev)** / **DeepSeek Harness** / **Cordis** / **HERMES** / **Honcho** / **n8n** / **Overleaf** — 詳細見 [docs/reference.md](docs/reference.md)。

## 狀態

- 📝 設計階段（2026-08-14 起）
- 🔍 持續調查參考專案（deepwiki / gh 實證）
- 🏗️ 後端 / 前端目錄為空，待設計定稿後開發

## 相關

- Org: [github.com/drill-research-lab](https://github.com/drill-research-lab)
- Fork 參考: [deepseek-harness](https://github.com/drill-research-lab/deepseek-harness)
