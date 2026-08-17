# Track A：PI-based（備用 / 個人線）

> [decisions.md](../decisions.md) D1 雙軌之一。目標：以 PI 生態系（pi + pi-* plugins）組合出 Drill 的全部功能——同時是 Track B 的參照實作與備援。
> Plugin 複用清單：[reference/pi-ecosystem.md](../reference/pi-ecosystem.md)。

## 複用地圖（feature → 複用 / gap）

| Feature | 主要複用 | Gap（需自建） | 詳情 |
|---|---|---|---|
| normal | pi-subagents（派遣 + workflowScript）、omo 的 agent 定義 schema（抄設計） | `shared-project` context mode、Drill agent 定義集、活動指示器 UI | [features/normal/pi.md](../features/normal/pi.md) |
| wiki | pi-knowledge（檢索）、markitdown/anydoc（解析）、pi-arxivist/pi-web-access（arXiv/影片） | 多本 notebook 資料模型、組織層（llm-wiki 三層）、`kb_link`/`kb_summarize` tools | [features/wiki/pi.md](../features/wiki/pi.md) |
| pipeline | pi-dynamic-workflows（fan-out/journal resume 參考）、pi-subagents（節點執行語意）、n8n 設計（抄不 embed） | workflow engine 四件（schema/FSM/scheduler/MCP expose）、builder tool schema、DAG editor | [features/pipeline/pi.md](../features/pipeline/pi.md) |
| research | PI coding agent 工具面、Gondolin/pi-landstrip（沙盒）、git-checkpoint（回溯） | `run_experiment`/`analyze_results` tools、編輯器前端 | [features/research/pi.md](../features/research/pi.md) |
| paper | pi-paper-lab、pi-bib、pi-arxivist、pi-research-agent、paper-pilot MCP（Zotero） | writer tools 包裝、LaTeX compile service、編輯器、模板倉 | [features/paper/pi.md](../features/paper/pi.md) |
| review | pi-subagents fresh mode（clean-context）、pi-web-access、pi-bib、/deep-research 模式 | 驗證分級狀態機、rubric agents、claim 標記 UI | [features/review/pi.md](../features/review/pi.md) |
| skills | PI 原生 SKILL.md、`pi install`（npm/git）、pi-hermes-memory 的 skill tool | 管理 UI、自動演化+審批+Curator、skill-embedded MCP | [features/skills/pi.md](../features/skills/pi.md) |
| mcp | **pi-mcp-adapter**（完整 bridge：proxy/direct tools、OAuth、4 層 config） | 管理 UI、pipeline servers 動態註冊、per-agent 權限 | [features/mcp/pi.md](../features/mcp/pi.md) |
| setting | pi-ai（providers/keys/fallback）、permission-gate 模式 | 設定 UI+config API、多使用者/LDAP、專案管理、通知 | [features/setting/pi.md](../features/setting/pi.md) |
| system-status | @braintrust/pi-extension（tracing）、pi session log、pi-ai usage | metrics 聚合、WebSocket 推送、dashboard、per-user 用量（D3） | [features/system-status/pi.md](../features/system-status/pi.md) |

## 橫切組合（多功能共用）

| 主題 | 複用 | 使用者 |
|---|---|---|
| LLM provider 抽象 | pi-ai（47 providers、spark `:8000/v1` 走 OpenAI-compatible） | 全部 agent |
| Subagent 派遣 | pi-subagents | normal / review / pipeline 節點 |
| MCP 接入 | pi-mcp-adapter | mcp 頁 / paper（paper-pilot）/ wiki（KB MCP）|
| 檢索引擎 | pi-knowledge | wiki / normal 記憶（共用選型） |
| Web 研究 | pi-web-access | review / wiki / normal |

## 組合架構

pi-agent-core 之上需自建的 runtime 骨架（每功能 gap 的聯集）：
1. **服務化外殼**：pi 是 CLI/agent toolkit，無常駐 web 服務——需包一層 server（pi `--mode rpc` / `pi-server` 是候選底座）承載 10 個功能的後端 API
2. **Workflow engine**（pipeline 主體，見 [features/pipeline/pi.md](../features/pipeline/pi.md)）
3. **Web UI 殼**：10 個功能頁 + WebSocket（技術棧未定）
4. **用戶/workspace 層**：pi 是 session-per-cwd 無多用戶概念——必須功能，自建（D3）
