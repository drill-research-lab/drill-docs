# Drill Product Surfaces

> 產品命名基線見 [Product Vocabulary](product-vocabulary.md)。`normal/wiki/pipeline/research/paper` 等資料夾名稱暫時保留為 legacy internal paths。

主要頁面：

- Chat
- Library
- Pipelines
- Lab
- Writing（暫定）
- Review
- Notebook
- Files

設定／管理：

skills
mcp
Settings
System Status

## 跨功能設計

- [Product Vocabulary](product-vocabulary.md)：產品頁面、物件、agent 與 Agent / Mode selector 命名基線
- [Workspace / Project 所有權模型](workspace-project-ownership.md)：Workspace = workstation；Project = 有目標與生命週期的研究工作
- [Conversation Note / Notebook](conversation-note.md)：每個 Conversation 一份 Note；Notebook 集中瀏覽 Notes


## Chat（legacy path: `normal`）

主要 Conversation 與 agent 互動入口。

like chatgpt/gemini web
但我想讓他是 hermes agent

可以 call 其他 agent as subagent 作任何事情
像是 call librarian 某個知識庫的內容
或是 call reviewer 驗證事實

📄 詳細設計 → [`features/normal/spec.md`](features/normal/spec.md)

## Library（legacy path: `wiki`）

類似 NotebookLM 或 llm-wiki，但產品名稱使用 Library，與 Conversation Notes 的 Notebook 分開。
使用者可以上傳任意文件，llm 會自動協助與整理，可能會使用 anydoc 或是 markitdown 或是 ocr
RAG ?

AGENT: librarian

📄 詳細設計 → [`features/wiki/spec.md`](features/wiki/spec.md)

## Pipelines（legacy path: `pipeline`）

可以建立 pipeline 與 cronjob 
就像是 n8n 在做的事情
使用者或是 pipeline builder 可以設定 cronjob 定期執行或是被 toolcall 觸發 (as mcp)
每個節點可以是程式碼/agent/llm
建立完之後可以被 agent 呼叫 (like mcp) 或是 cronjob 觸發

AGENT: 可以 multistep 執行（設定 prompt/skills/tools），完成後傳給後續節點
llm: single llm ask 然後傳給後續節點

AGENT: pipeline builder / mini agent / simple llm

📄 詳細設計 → [`features/pipeline/spec.md`](features/pipeline/spec.md)

## Lab（legacy path: `research`）
可以建立實驗或是 poc
可以編輯程式碼
llm 生成的文件或程式碼可以讓用戶點開在右邊查看，框起來 suggest fix 或編輯
也可以編輯任意程式碼
就像 coding agent (claude/codex/opencode)
用起來會像是 vscode + copilot

AGENT: researcher

📄 詳細設計 → [`features/research/spec.md`](features/research/spec.md)

## Writing（legacy path: `paper`；名稱暫定）
跟 agents 一起建立 paper
md or latex
自帶 latex builder / preview / pdf viewer
就像 overleaf 一樣

AGENT: writer

📄 詳細設計 → [`features/paper/spec.md`](features/paper/spec.md)

## Review（legacy path: `review`）

review 產生的結果
或是 review 其他東西

也可以被其他 agent 呼叫 ，在無污染上下文的情況驗證事實，anti 幻覺

📄 詳細設計 → [`features/review/spec.md`](features/review/spec.md)

## Notebook

集中瀏覽每個 Conversation 的 Conversation Note，不是第二套 Wiki。

📄 詳細設計 → [`conversation-note.md`](conversation-note.md)

## Files

Workspace / Project 的檔案與產出物入口；所有權見 Workspace / Project 模型。

📄 詳細設計 → [`workspace-project-ownership.md`](workspace-project-ownership.md)

## skills 
add import edit skills files

📄 詳細設計 → [`features/skills/spec.md`](features/skills/spec.md)

## mcp
add import edit mcps 

📄 詳細設計 → [`features/mcp/spec.md`](features/mcp/spec.md)

## setting page

just any setting

📄 詳細設計 → [`features/setting/spec.md`](features/setting/spec.md)

## system status

system status
backend
frontend
llm
network
storage..etc

📄 詳細設計 → [`features/system-status/spec.md`](features/system-status/spec.md)

---

## 文件索引

| 文件 | 內容 |
|---|---|
| [tracks/pi.md](tracks/pi.md) | **Track A（PI-based）**：複用地圖、自建 gap、組合架構 |
| [tracks/dsh.md](tracks/dsh.md) | **Track B（DSH-based，團隊主線）**：fork 狀態、seam↔功能對應 |
| [reference/](reference/) | 外部參考專案調查（通用，與軌道無關）：總覽＋命名陷阱 |
| [reference/dsh.md](reference/dsh.md) | DeepSeek Harness 深度設計調查（15 面向 + 設計決策輸入，commit 級驗證） |
| [concepts.md](concepts.md) | Agent 工程術語階梯：prompt / context / harness / loop / graph engineering（2026-08 基準） |
| [decisions.md](decisions.md) | 專案決策紀錄（雙軌策略、自進化、storage 定義、部署、分工、開源政策） |
| [features/](features/) | 功能區導覽（[features/index.md](features/index.md)）：每個功能一個 subfolder — spec.md（共同契約）+ pi.md（Track A 設計）+ dsh.md（Track B 註記） |
| [archive/disscuss.md](archive/disscuss.md) | ⚠️ 已棄用——早期 Spark 階段討論，僅存歷史（已遷至 archive/） |
| [art.md](art.md) | 品牌 / 命名（鑽頭、天元突破） |
