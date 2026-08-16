# Reference — 分域調查

> 各領域的外部專案調查（與軌道無關）。DSH 深度 → [dsh.md](dsh.md)；Cordis paper → [cordis.md](cordis.md)；PI 生態系 → [pi-ecosystem.md](pi-ecosystem.md)。

---

## 1. Coding Agent（主要參考）

| 名稱 | 連結 | 授權 | 一句話介紹 |
|---|---|---|---|
| **OpenCode** (MAIN) | [github.com/anomalyco/opencode](https://github.com/anomalyco/opencode) · [opencode.ai](https://opencode.ai) | OSS (MIT) | 開源 AI coding agent（TypeScript terminal-first，含 TUI、web console、IDE extension、SDK） |
| **oh-my-openagent** (MAIN, 舊名 oh-my-opencode) | [github.com/code-yeongyu/oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent) · [omo.dev](https://omo.dev) | OSS | 多 agent 編排 harness，把 OpenCode/Codex/Senpi 變成「開發團隊」 |
| **oh-my-pi** | [github.com/can1357/oh-my-pi](https://github.com/can1357/oh-my-pi) · [omp.sh](https://omp.sh) | OSS | 獨立終端 AI coding agent，fork 自 Pi（by mariozechner） |

### OpenCode 子 agent 派遣機制（task tool）

- `TaskTool` 參數：`description` / `prompt` / `subagent_type` / `task_id`（resume）/ `command` / `background`（async）
- 子 session 記錄 `parentID`；`BackgroundJob.Service` 完成時 `injectBackgroundResult` 送合成訊息回 parent
- Permission 繼承（DeepWiki 實證 2026-08-16）：child 繼承 parent 的 `deny` + `external_directory`（**硬上限**，子定義無法覆蓋）；預設 deny `todowrite`/`task`（除非 subagent 自身 ruleset 明確允許）；但 subagent 自身的 `allow` 規則**可以比 parent agent 更鬆**（last-matching-rule wins，tests 證實）
- Config 來源：`opencode.json` 的 `agent` key、或 `.opencode/agents/*.md`（frontmatter + body 即 prompt）
- 深度限制：`subagent_depth` 預設 **1**（primary → subagent，subagent 不能再開）
- 同步 = 預設；async 需 `OPENCODE_EXPERIMENTAL_BACKGROUND_SUBAGENTS=true`
- 結果回傳 = 最終文字（非全 token stream），XML-like 包裝（sessionID / state / summary）

### oh-my-openagent omo.jsonc schema（agent 定義）

Top-level keys：`$schema` / `categories` / `agents` / `codegraph` / `task` / `teams` / `models` / `memory` / `telemetry` / `[opencode]` / `[senpi]` / `[codex]` / `profiles` / `_migrations`

Agent 定義（`OmoAgentDefSchema`）：
```jsonc
{
  "description": "...",
  "prompt": "...",           // 或 file:// 載入
  "model": "...",            // 或 models: 有序 fallback chain
  "reasoning": "max|high|low",
  "tools": { "toolName": true },   // 允許的工具
  "execution_mode": "in-process|process",
  "background": false,
  "max_depth": 2,            // 子 agent 委派深度
  "allowed_subagents": ["librarian", "reviewer"],
  "disallowed_tools": [...],
  "max_turns": 50,
  "temperature": 0.7,
  "disable": false
}
```

- **Category vs Agent**：`task(category="quick")` → 走 `sisyphus-junior` + category 設定；`task(subagent_type="librarian")` → 直接指定 agent。category 是「語意任務域」的 model/tool preset
- **Team mode 儲存**（`~/.omo/teams/{name}/` 或 `<project>/.omo/teams/{name}/`）：
  - `config.json` — TeamSpec（name / description / lead / members）
  - `state.json` — durable runtime state（members / session IDs / lifecycle）
  - `mailbox/` — 每位 member 一個 `.jsonl` inbox；`team_send_message` tool 送訊；poller 取未讀注入 session
  - `tasklist.jsonl` — 共享任務清單
  - `worktrees/` — 每位 member 的 git worktree（隔離 code 變更）
- **Harness adapter**：OpenCode（full）/ Codex（輕量）/ Senpi（native）三種 harness，共用 omo.jsonc，adapter 是 thin wrapper
- **Three-tier MCP**：built-in（`packages/omo-opencode/src/mcp/`）+ Claude Code（`.mcp.json` project/user）+ skill-embedded（SKILL.md frontmatter 宣告）
---

## 2. Agent Framework / Platform

| 名稱 | 連結 | 授權 | 一句話介紹 |
|---|---|---|---|
| **PI** (pi.dev) | [github.com/earendil-works/pi](https://github.com/earendil-works/pi) · [pi.dev](https://pi.dev) | OSS (MIT) | Agent toolkit；`pi-ai` 是統一 LLM provider 抽象層 |
| **DeepSeek Harness** (DSH) | [github.com/deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness) · [deepseek.com/harness](https://deepseek.com/harness) | OSS (MIT) | DeepSeek 官方 agent harness（Cordis plugin kernel — "everything is a plugin"）；multi-agent subagent、Web UI、ACP (JSON-RPC stdio)、Headless、Python/TS SDK、append-only session log |
| **HERMES Agent** | [github.com/NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) · [hermes-agent.nousresearch.com](https://hermes-agent.nousresearch.com) | OSS | Self-improving AI agent with long-term memory（Honcho 整合） |
| **Odysseus** | [github.com/odysseus-dev/odysseus](https://github.com/odysseus-dev/odysseus) · [odysseus-dev.github.io/odysseus](https://odysseus-dev.github.io/odysseus) | OSS (AGPL-3.0) | 自架 local-first AI workspace（chat + agents + RAG + MCP + email/calendar/docs） |

DSH 深度調查 → [dsh.md](dsh.md)（15 面向、commit 級驗證）

### HERMES Agent 詳細

- **MemoryProvider ABC**（`agent/memory_provider.py`）：`name` / `is_available` / `initialize(session_id)` / `system_prompt_block` / `prefetch` / `sync_turn` / `get_tool_schemas` / `handle_tool_call` / `shutdown`
- **內建 providers**（`plugins/memory/`）：Honcho / OpenViking / Hindsight（knowledge graph）/ Mem0 / RetainDB / ByteRover / SuperMemory / Holographic；一次只能啟用一個 external provider
- **Skill 系統**（DeepWiki 實證 2026-08-16，兩輪查詢）：
  - **結構**：`~/.hermes/skills/<name>/` — `SKILL.md`（YAML frontmatter：`name`/`description` 必填，`platforms`/`prerequisites`/`version`/`license`/`metadata` 選填）+ `references/` / `templates/` / `scripts/` / `assets/` 子目錄；安裝後自動成為 slash command
  - **Progressive disclosure 三層**：L0 `skills_list()`（輕量 index，全庫約 ~3k tokens）→ L1 `skill_view(name)`（載入全文）→ L2 `skill_view(name, path)`（references/ 等支援檔按需載入）
  - **`skill_manage` tool**：`create` / `edit`（全文替換）/ `patch`（find-replace，含支援檔）/ `delete` / `write_file` / `remove_file`；操作後清 skills prompt cache、更新 usage telemetry、debounced sync push
  - **自動演化**：前景 agent 用 `skill_manage` 自我改善；背景 review agent 由 `_SKILL_REVIEW_PROMPT` 驅動。觸發：user corrections / non-trivial techniques（fix、workaround、debugging path、tool-usage pattern）/ outdated skills。優先序：改已載入 skill → 改 umbrella skill → 建 class-level skill
  - **寫入審批**：`skills.write_approval`（預設 `false` 自由寫）；`true` 時所有寫入 staged 到 `~/.hermes/pending/skills/`，`/skills pending|diff|approve|reject|approval on|off` 審批
  - **Protected skills**（不可被自演化改）：bundled / hub-installed / `external_dirs` / pinned / user-owned
  - **Curator（背景維護）**：閒置觸發（距上次 ≥ `interval_hours` 7 天 + agent 閒置 ≥ 2 小時才 fork 背景 `AIAgent`），非 cron。只管 agent-created skills（`.usage.json` 標記）。動作：30 天未用標 stale、90 天未用 archive（移到 `.archive/`，可恢復，是其**最大**破壞性動作、不自動刪）、LLM consolidation（預設關，`curator.consolidate: true` 開）把重複 skill 合併成 umbrella。每次 pass 前 tar.gz 快照可 rollback。指令：`pin` / `unpin` / `adopt` / `restore` / `archive` / `prune` / `backup` / `rollback` / `pause` / `resume` / `status` / `run --dry-run`
  - **三重 guard**：`_pinned_guard`（pinned 不可 delete，但可 patch/edit）；`_background_review_write_guard`（背景寫入比前景嚴——沒有 user 在場同意）；`_curator_consolidation_delete_guard`（consolidation 的 delete 只允許「內容已被 umbrella 吸收」）
- **Honcho 整合**：tool `honcho_profile` / `honcho_search` / `honcho_context` / `honcho_reasoning` / `honcho_conclude`；`<memory-context>` fenced injection
---

## 3. Workflow Engine

| 名稱 | 連結 | 授權 | 一句話介紹 |
|---|---|---|---|
| **n8n** | [github.com/n8n-io/n8n](https://github.com/n8n-io/n8n) · [n8n.io](https://n8n.io) | OSS (fair-code) | Workflow automation，內建 LangChain AI nodes |

### n8n 詳細

- **Node types**：`n8n-nodes-base.manualTrigger`（trigger）/ `@n8n/n8n-nodes-langchain.agent`（AI agent）/ `@n8n/n8n-nodes-langchain.lmChatOpenAi`（LLM）；type = 字串識別
- **Triggers**：cron / webhook / chat / form 都是 trigger node types；scheduler 在 `n8n-core`；workflow 定義存 DB，重啟後重新註冊
- **Expose as API**：webhook trigger → `{webhookBaseUrl}/{path}`；**MCP server mode**：`McpService` 暴露工具（create/update/execute/validate workflow）
- **Programmatic build**：`@n8n/workflow-sdk` — `workflow().add(startTrigger).to(aiNode)`；`generateWorkflowCode` / `parseWorkflowCode` 轉 `WorkflowJSON`（可 LLM 生成）；`n8n-workflow` 提供核心 types
- **Data flow**：items = JSON 陣列；batch 傳輸（非 token stream）；`AgentRuntime` 可用 AI SDK `generateText`/`streamText`
- **Embed 限制**：core engine 依賴 TypeORM（SQLite/Postgres）+ DI；無現成「純 library 執行」單一 API（`workflow-executor.ts` 在 evaluation context）；license 是 Sustainable Use License（fair-code，商用有門檻）

---

## 4. Document Processing

| 名稱 | 連結 | 授權 | 一句話介紹 |
|---|---|---|---|
| **anydoc** | [github.com/firecrawl/anydoc](https://github.com/firecrawl/anydoc) · [firecrawl.github.io/anydoc](https://firecrawl.github.io/anydoc/) | OSS (MIT) | Rust 函式庫，把 Word/PPT/Excel/PDF 等轉成 clean Markdown（GFM） |
| **markitdown** | [github.com/microsoft/markitdown](https://github.com/microsoft/markitdown) | OSS (MIT) | Python 工具（CLI + lib + MCP server），轉換 20+ 格式到 Markdown |

- **anydoc**：Rust core（<5ms median）、parse-model-render pipeline、Node (napi-rs) / Python (PyO3) bindings；**無 OCR**（scanned PDF 不支援）
- **markitdown**：Python、20+ formats（office / PDF / HTML / media / MSG / EPUB / ZIP）、priority converter registry、plugin system、`markitdown-mcp` + `markitdown-ocr`（LLM-vision OCR）子套件；**OCR 需 markitdown-ocr plugin**

---

## 5. Editor / IDE

| 名稱 | 連結 | 授權 | 一句話介紹 |
|---|---|---|---|
| **VS Code** | [github.com/microsoft/vscode](https://github.com/microsoft/vscode) · [code.visualstudio.com](https://code.visualstudio.com) | OSS (MIT) | Microsoft 開源 Electron-based 編輯器 |
| **Overleaf** | [github.com/overleaf/overleaf](https://github.com/overleaf/overleaf) · [overleaf.com](https://www.overleaf.com) | OSS (AGPL-3.0) | 開源 web-based 即時協作 LaTeX 編輯器（Community Edition 可自架） |
| **GitHub Copilot** | [github.com/features/copilot](https://github.com/features/copilot) · docs [github/copilot-docs](https://github.com/github/copilot-docs) | CLOSED | GitHub 閉源 AI coding assistant |

### Overleaf 詳細（架構調查）

- **Realtime**：`services/real-time/`，socket.io（WebSocket + xhr-polling fallback）；OT-based（operation-based，非 CRDT）
- **CLSI compile**：`services/clsi/`；`CompileController` → `CompileManager` → `ClsiManager` → CLSI service（Docker container 隔離執行 LaTeX）→ PDF cache（`ClsiCacheController`）
- **Storage**：Docstore（chunk = snapshot + changes，Postgres `chunk_store`）；DocumentUpdater（apply ops + flush to Mongo）；history-v1（版本歷史 / time-travel / RestoreManager）
- **注意**：deepwiki 未確認底層 OT library（ShareDB/ot.js/custom）；embed 需要 real-time + document-updater + docstore + history 四服務，獨立性低 → **自建 Yjs-based editor 是替代方案**

---

## 6. Knowledge Base / Notes

| 名稱 | 連結 | 授權 | 一句話介紹 |
|---|---|---|---|
| **llm-wiki** | [github.com/nashsu/llm_wiki](https://github.com/nashsu/llm_wiki) | OSS | 桌面 app（Tauri+Rust+React），把文件變成 persistent 互連 wiki |
| **NotebookLM** (已於 2026-07 改名 **Gemini Notebook**) | [notebooklm.google.com](https://notebooklm.google.com/) | CLOSED | Google 的 source-grounded AI research assistant |
| **Obsidian** | [obsidian.md](https://obsidian.md/) · community repo [github.com/obsidianmd/obsidian-releases](https://github.com/obsidianmd/obsidian-releases) | CLOSED | 免費 local Markdown 筆記 app，強大 plugin 生態 |

### llm-wiki 詳細

- Tauri v2（Rust backend + React 19 frontend）
- **三層模型**：raw sources / LLM-generated wiki Markdown / schema
- 非 RAG：LLM 增量建立並維護 persistent wiki；vector search + knowledge-graph view + Rust chat agent

---

## 7. Consumer AI / Reference Products

| 名稱 | 連結 | 授權 | 一句話介紹 |
|---|---|---|---|
| **ChatGPT** | [chatgpt.com](https://chatgpt.com/) | CLOSED | OpenAI 的消費級 AI 助理 |
| **Gemini** | [gemini.google.com](https://gemini.google.com/) · OSS CLI [google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli) | CLOSED | Google 的消費級 AI 助理 |
| **Gemini DeepResearch** | [gemini.google/overview/deep-research](https://gemini.google/overview/deep-research/) | CLOSED | Gemini 內建 agentic feature，自動瀏覽上百網站產出 cited research report |
| **Claude** | [claude.ai](https://claude.ai/) · OSS CLI [anthropics/claude-code](https://github.com/anthropics/claude-code) | CLOSED | Anthropic 的消費級 AI 助理 |
| **Codex** | [github.com/openai/codex](https://github.com/openai/codex) · [chatgpt.com/codex](https://chatgpt.com/codex) | OSS (Apache-2.0) | OpenAI coding agent（Rust CLI + 雲端 Codex Web） |
| **Perplexity** | [perplexity.ai](https://www.perplexity.ai/) | CLOSED | AI-powered answer engine，含 citations |

---

## 8. Protocol / Model

| 名稱 | 連結 | 授權 | 一句話介紹 |
|---|---|---|---|
| **MCP** (Model Context Protocol) | [github.com/modelcontextprotocol/modelcontextprotocol](https://github.com/modelcontextprotocol/modelcontextprotocol) · [modelcontextprotocol.io](https://modelcontextprotocol.io) | OSS (MIT) | 開放協議標準，讓 AI app 透過 JSON-RPC 2.0 連接外部 tools/data |
| **DeepSeek-V4-Flash-0731** | [huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731](https://huggingface.co/deepseek-ai/DeepSeek-V4-Flash-0731) | OSS (MIT) | DeepSeek-V4-Flash 官方版本（304B params MoE，含 DSpark speculative decoding） |

### DeepSeek-V4 系列（官方 paper, arXiv 2606.19348）

- **V4-Pro**：1.6T params / 49B activated；**V4-Flash**：284B params / 13B activated（官方 paper 數字，非 HF card 的 304B——後者含 DSpark draft module）
- 1M token context；hybrid attention（CSA + HCA）；mHC residual；Muon optimizer
- 1M context 下 V4-Pro 只需 V3.2 的 27% FLOPs / 10% KV cache；Flash 只需 10% / 7%
- FP4 quantization-aware training（routed expert weights + indexer QK path）
- Agentic capabilities：「DeepSeek-V4-Pro-Max 勝過 Claude Sonnet 4.5、接近 Opus 4.5」（官方內部 eval）

### MCP 詳細

- 開放協議：AI apps ↔ external data/tools；JSON-RPC 2.0；stdio / Streamable HTTP / SSE transports
- Capabilities：server（tools / resources / prompts）+ client（sampling / elicitation / logging）

---

## 9. Honcho（memory layer 選項）

| 項目 | 內容 |
|---|---|
| **Repo** | [github.com/plastic-labs/honcho](https://github.com/plastic-labs/honcho) |
| **License** | AGPL-3.0 |
| **部署** | Managed（api.honcho.dev）或自架 FastAPI server（Docker） |
| **核心概念** | **Peer-centric**：user / agent / group / project 都是 peer；Workspace → Peer → Session → Message |
| **Reasoning-first** | 背景非同步推理（Deriver）：fact derivation / session summaries / peer cards / dialectic reasoning；Neuromancer 專用推理模型 |
| **API** | `session.add_messages` / `peer.chat`（dialectic）/ `session.context().to_openai()/.to_anthropic()`（prompt-ready）/ `peer.search`（hybrid BM25+vector）/ `peer.representation`（低延遲）/ `session.upload_file` |
| **多視角** | 可配置 local（peer A 對 peer B 的模型）vs global representation |
| **對 Drill** | 適合「user 建模 + 跨 session 記憶」；AGPL 注意；若只要 agent 自身記憶，pi-hermes-memory 或 pi-knowledge 已足夠 |
