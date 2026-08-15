# Reference — Drill 參考專案總覽

> Drill 設計中提及的所有外部專案/產品清單與調查結果。
> 標記：**OSS** = 開源可自架/fork；**CLOSED** = 閉源商業產品；**MAIN** = Drill 主要參考對象。

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
- Permission 繼承：child 繼承 parent 的 `deny` + `external_directory`；預設禁止 `todowrite`/`task`；child 不能超過 parent 權限（可更鬆的規則，測試證實）
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

### PI (pi.dev) 詳細

- **pi-ai**：47 個 provider factory（Anthropic / OpenAI / Google / Bedrock / Mistral / Groq / xAI / HuggingFace / Kimi / MiniMax / NVIDIA / OpenRouter / Ollama / llama.cpp…）
- **pi-agent-core**：`Agent` class（initialState / streamFn / tools / transformContext / beforeToolCall / afterToolCall / shouldStopAfterTurn）+ `agentLoop()` 低階 API；`AgentTool<TParameters, TDetails>`（schema + execute(parameters, context, signal)）
- **Session backends**：`JsonlSessionRepo`（預設）+ `@earendil-works/pi-session-backend-sqlite-node`；`SessionRepo` 是 interface 可自接 Postgres/Redis
- **Extension system**：TS module 匯出 default factory（`ExtensionAPI`）；`pi.on()` 事件（session_start / before_agent_start / message_* / turn_* / tool_call / tool_result / user_bash / session_shutdown）；`pi.registerTool()` / `pi.registerCommand()` / `pi.registerProvider()`；`ctx.ui` 互動
- **In-tree extensions**（`packages/coding-agent/examples/extensions/`）：permission-gate、todo（stateful tool）、custom-provider-anthropic、custom-compaction、git-checkpoint、plan-mode（完整實作）、ssh、interactive-shell、sandbox、gondolin（micro-VM）、subagent、event-bus、session-name…
- **安裝**：`pi install npm:@foo/bar`（~/.pi/agent/npm/）或 `pi install git:github.com/...`；`jiti` runtime 直接跑 TS
- **pi.dev/packages**：npm keyword `pi-package` 的 gallery；package.json `pi` field（extensions / skills / prompts / themes）
- **RPC mode**：`pi --mode rpc`（stdin/stdout JSON protocol）；`pi-server`（experimental，session server over CBOR/Unix socket）；`pi-protocol` 定義 wire DTO
- **Sandbox**：Gondolin（micro-VM 路由工具）/ Docker / OpenShell（policy-controlled）
- **無內建**：MCP（by design）、memory provider、cron、multi-tenancy、web UI、RAG primitive

### DeepSeek Harness (DSH) 詳細

- **Cordis plugin kernel**：plugin 提供 services / typed events / reversible effects 到共享 Context；"everything is a plugin"
- **Capability Seams**：每個可替換能力 = service definition + provider + consumer
  - 例：`ctx.subagents` seam → providers：`spawn-in-process` / `fork-in-process` / `acp` / `codex` / `claude-code` / `dsh-sdk`
  - 例：`ctx.llm` seam → providers：`dsh-llm-deepseek`（原生）/ `dsh-llm-pi-ai`（可互操作 PI 的 47 providers）
- **Profiles + Bundles**：profile = 命名 runtime 組合（web / headless 兩種模板）；bundle = 可發佈的 plugin 群組；`cordis.patch.yml` 層疊設定
- **Modes**：Standard（full toolset）/ Code（model 寫 TS orchestrate）/ Minimal（bash + str_replace_editor，benchmark 用）/ Creator（runtime 檢視 + plugin 測試）
- **Session log**：append-only `SessionEvent`（system prompt / reasoning / tool calls / subagent scheduling / context injection 全記錄）；`ctx.sessionQuery` seam（FTS5 / lineage / bounded reads / semantic filter）；可 resume / fork / search / replay；Trajectory view 按 source 檢視
- **Subagent depth**：`delegationDepth` monotone（只能增加）；`maxSubagentDepth` 預設限制
- **Embedding**：`@deepseek-ai/dsh-sdk-client`（TS）+ Python SDK，stdio JSON-RPC 驅動 runtime
- **MCP**：`dsh-mcp-client` 可消費（plugin per server 註冊 tools）；不預設啟用

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

## 9. PI 生態系 Plugins（調查結果：能用就複用）

### 核心擴充（高度相關）

| Plugin | 連結 | 用途 |
|---|---|---|
| **pi-mcp-adapter** | [github.com/nicobailon/pi-mcp-adapter](https://github.com/nicobailon/pi-mcp-adapter) | MCP bridge：`mcp` proxy tool + direct tools promotion；stdio / StreamableHTTP / SSE；OAuth 2.1（`~/.pi/agent/mcp-oauth/`）；`.mcp.json` 4 層 config（user-global / pi-global / project / pi-project）；lazy connect + idle timeout；output guard（truncate 大輸出） |
| **pi-subagents** | [github.com/nicobailon/pi-subagents](https://github.com/nicobailon/pi-subagents) | 子 agent 派遣：builtin agents（scout / researcher / worker / reviewer / oracle / delegate）；agent = MD + YAML frontmatter；`subagent()` tool；`workflowScript`（JS：`runs.run` / `runs.all` 平行 / branching / retry / gate / aggregation）；context 模式（fresh / fork）；`contact_supervisor`（child→parent mid-execution）；maxSubagentDepth 預設 2 層；worktree isolation |
| **pi-web-access** | [github.com/nicobailon/pi-web-access](https://github.com/nicobailon/pi-web-access) | Web research tools：`web_search`（multi-query / recency / domain filter）、`code_search`（Exa）、`fetch_content`（URL → Markdown；自動處理 GitHub clone / YouTube / PDF / 本地影片）、`get_search_content`；providers：Exa / Perplexity / Gemini API / Gemini Web fallback chain；PDF 用 unpdf（無 OCR）；YouTube 用 Gemini multimodal |
| **pi-hermes-memory** | [github.com/chandra447/pi-hermes-memory](https://github.com/chandra447/pi-hermes-memory) | HERMES memory 移植到 PI：L1 MEMORY.md/USER.md、L2 Skills、L3 SQLite、L4 FTS5 session search；tools：`memory`（add/replace/remove）、`skill`（create/view/patch/edit/delete）、`memory_search`、`session_search`；scope：global / project / user；auto-consolidation；**無 vector search**（只有 FTS5）；Honcho/Mem0 外部 providers deferred |

### 記憶 / RAG 家族（選擇題）

| Plugin | 連結 | 特色 |
|---|---|---|
| **pi-knowledge** | [github.com/nczz/pi-knowledge](https://github.com/nczz/pi-knowledge) | Local-first RAG KB：BM25 + semantic vectors + weighted fusion；cross-encoder rerank；code-aware chunking（TS/JS/Py/Go/Rust/Java）；symbol lookup；local embeddings（零 API key）；`~/.pi/knowledge/` |
| **pi-memory** | [github.com/jayzeng/pi-memory](https://github.com/jayzeng/pi-memory) | 最 popular：daily logs + scratchpad + MEMORY.md；qmd 提供 keyword/semantic/deep 三模式搜尋 |
| **@pi-unipi/memory** | [npm](https://www.npmjs.com/package/@pi-unipi/memory) | MemPalace backend（vector search）+ SQLite fallback；project/global 兩 tier |
| **pi-agent-memory** | [npm](https://www.npmjs.com/package/pi-agent-memory) | claude-mem fork（55k stars）：cross-engine 共享記憶；FTS5 + Chroma hybrid |
| **pi-semantic-memory** | [npm](https://www.npmjs.com/package/pi-semantic-memory) | LogosDB MCP server：local-first vector search、stdio-only、每 turn 自動注入 |
| **@arvoretech/pi-memory** | [npm](https://www.npmjs.com/package/@arvoretech/pi-memory) | Cloud RAG（Qdrant + GitHub OAuth）：org-wide 共享記憶、raw/curated 兩 tier |

### 學術研究工具（paper mode 直接可用）

| Plugin | 連結 | 用途 |
|---|---|---|
| **Aspis0/pi-paper-lab** | [github.com/Aspis0/pi-paper-lab](https://github.com/Aspis0/pi-paper-lab) | Paper writing：anti-AI rewrite、Vancouver citations、.docx output（Word-native citation fields）；citation backends：CrossRef / Serper / Exa / OpenAlex / Europe PMC |
| **hinsencamp/pi-research-agent** | [github.com/hinsencamp/pi-research-agent](https://github.com/hinsencamp/pi-research-agent) | 學術搜尋 skill：Semantic Scholar + arXiv + OpenAlex（免 API key）；搜尋/抓取/全文本（arXiv）/綜整 ranking |
| **portos-wang/pi-extensions** | [github.com/portos-wang/pi-extensions](https://github.com/portos-wang/pi-extensions) | Claude Code → PI 移植 hub：**39-agent academic pipeline**（Deep Research 13-agent / Academic Paper 12-agent / Reviewer 7-agent / Pipeline 10-stage orchestrator） |
| **pi-bib** | [npm](https://www.npmjs.com/package/pi-bib) | BibTeX review：CrossRef + Semantic Scholar 驗證、duplicate 偵測、safe suggested files |
| **@wienerberliner/pi-arxiv** | [npm](https://www.npmjs.com/package/@wienerberliner/pi-arxiv) | arXiv 搜尋 / lookup / Markdown fetch（arxiv2md） |
| **pi-arxivist** | [npm](https://www.npmjs.com/package/pi-arxivist) | arXiv source → Markdown via pandoc WASM（零系統依賴、math 保留） |
| **aytzey/paper-pilot** | [github.com/aytzey/paper-pilot](https://github.com/aytzey/paper-pilot) | **MCP server**（任何 MCP-capable agent 可用）：6 學術 DB 搜尋 + OA PDF 下載 + full-text 閱讀 + evidence extraction + 圖表渲染 + Zotero sync |
| **spignotti/academic-agent** | [github.com/spignotti/academic-agent](https://github.com/spignotti/academic-agent) | Zotero-centric：tag-scoped RAG + citation-ready writing；zotero-companion MCP server |

### Web research 家族

| Plugin | 連結 | 特色 |
|---|---|---|
| **pi-web-research** | [npm](https://www.npmjs.com/package/pi-web-research) | Kagi/Wyna backend；search → 平行 SSRF-guarded fetch → Readability → MD → **isolated sub-agent distillation** → cited briefings；raw pages 不進主 context；prompt injection 防護 |
| **pi-fabric** | [github.com/monotykamary/pi-fabric](https://github.com/monotykamary/pi-fabric) | Programmable tool + agent runtime |
| **@quintinshaw/pi-dynamic-workflows** | [github.com/QuintinShaw/pi-dynamic-workflows](https://github.com/QuintinShaw/pi-dynamic-workflows) | Fan-out 100s subagents：real model routing、token/cost accounting、resume、git-worktree isolation、**/deep-research** |

### 其他值得注意

| Plugin | 用途 |
|---|---|
| **pi-intercom** | Agent↔agent / 跨 session 通訊 |
| **pi-background-tasks** | Durable background shell tasks + read-only delegated agents + child Pi processes |
| **pi-landstrip** | Sandboxed Bash + process-backed agents |
| **@gotgenes/pi-permission-system** | Permission enforcement |
| **@narumitw/pi-plan-mode** | Codex-like read-only /plan mode |
| **@braintrust/pi-extension** | Session/turn/LLM/tool 自動 tracing 到 Braintrust |
| **pi-agent-browser-native** | agent-browser 自動化原生 tool |
| **context-mode** | MCP plugin：context window 節省 + sandboxed code exec + FTS5 KB（跨 Claude/Gemini/Copilot/OpenCode/Codex） |
| **@vigolium/piolium** | Multi-phase security audits：specialist sub-agents、isolated context windows、resumable state |
| **@llblab/pi-telegram** | Telegram runtime adapter |

---

## 10. Cordis（DSH 的 plugin kernel）

| 項目 | 內容 |
|---|---|
| **Repo** | [github.com/cordiverse/cordis](https://github.com/cordiverse/cordis)（Koishi 團隊的 meta-framework） |
| **Paper** | [github.com/cordiverse/paper](https://github.com/cordiverse/paper) — *A Programming Paradigm for Spatiotemporal Composability*（北大 + DeepSeek，2026-08-13，preprint） |
| **核心概念** | **Temporal composability**（卸載 = 完全逆轉副作用）+ **Spatial composability**（宣告 + 反應式管理依賴） |
| **機制** | **Revertible effects**（每個 context transformation 帶 inverse，runtime 追蹤，`recover()` 逆序執行回初始態）+ **Reactive coeffects**（component 宣告依賴規格，context 變動時自動 notify activating/deactivating/neutral） |
| **形式化** | Calculus of dynamic composition；metatheory：Preservation / Temporal Composability / Spatial Composability / Progress / Confluence |
| **驗證** | Koishi chatbot：4 年、4000+ 社群 plugin、TypeScript |
| **設計啟示** | 比 PI extensions 更強：數學保證「裝得進去 = 拆得乾淨」；對 multi-agent（agent 互相 call）特別重要——卸載 agent 時 dependents 自動暫停 |

### 深度調查（完整讀完 paper 後）

**形式模型核心：**
- **Effect context** `∂Γ = Γ × (Γ→Γ)`：pair `(γ, φ)`，γ = 目前狀態，φ = accumulator（所有已執行 effect 的 inverse 合成）
- `track_Γ(f,g) = (γ,φ) ↦ (f(γ), φ∘g)` — 套用 forward map + 把 inverse 合成進 accumulator
- `recover_Γ = (γ,φ) ↦ (φ(γ), id_Γ)` — 套用 accumulator 回到初始態
- **Witnessed effect function** `𝔈*_Γ`：每個 effect 在套用點回傳 inverse，並帶 proof `g(δ)=γ`（inverse 真的還原）；Theorem 11：witnessing 在合成下封閉
- **Component**（formal）= triple `(d, p, e)`：d = 依賴規格（需要的 keys）、p = provision（提供的 keys）、e = witnessed effect function
- **單一來源紀律**：no two fibers 可提供重疊的 key → 一個 provider 一次只有一個 fiber

**5 個 metatheorem（保證什麼）：**
1. **Preservation**：10 條規則都保持 registry well-formed
2. **Temporal composability**：Recovery exactness — 套用 fiber 的 accumulator 只撤銷它自己的貢獻，不碰別人
3. **Spatial composability**：Ordering — provider 活得比 consumer 久；Resolution coherence — 每個 iteration 對固定 resolution 執行
4. **Progress**：無 deadlock（非 quiescent 必有規則可套）+ 終止（`S(n) ≤ (K+4)(V(n)+1)`）
5. **Confluence**：quiescent state 是 config 的純函數 — 任何 schedule 收斂到同一個「靜態組裝」normal form

**實作（Cordis 4）：**
- `ctx.effect(callback)` = **唯一 mutation primitive**，實作 effect iterator，回傳 `dispose()` closure
- `ctx.get/set`（keyed, typed）；`ctx.isolate(key, realm)` + `ctx.intercept(key, metadata)` = derived child context（無需 inverse）
- `ctx.use(component, config)` 實例化 fiber；`refresh/reload/unload` 是 inertial state machine（commit view → apply → 檢查 target → notify / chain unload）
- **Proxy access control**：`ctx[key]` 對照 fiber 的 committed view；undeclared → `UNDECLARED_ACCESS`；capability-style 的 load-time review
- **Loader**：entry = `(id, url, isolate, intercept, config, disabled)`；incremental reconciliation；`@cordisjs/group` / `@cordisjs/include` 是普通 component
- **HMR**：3 階段（import graph classification / stale detection / transactional reload with cache backup-rollback）；不需要 Webpack/Vite 的 annotation；ESM 無 public cache-eviction API 是 caveat
- **Koishi** 跑 Cordis v3；paper 呈現 v4（核心模型共享，semantics 精煉 + loader 重設計）

**誠實的限制（paper 自己承認）：**
1. **Inverse witness 沒有 runtime 檢查** — inverse 正確性是 component 作者的義務，runtime 不驗證（寫錯就 silent leak）
2. Recovery 是 **observational equivalence ≃**，不是 literal state equality（heap layout / names 不還原）
3. 保證**有前提**：所有共享位置綁在 commutative key、provision totality、`≺` acyclic、finite names
4. **Confluence 排除 failure**（failure 是真正 divergence source）
5. **Emissions（寫出/送出）無法 revert** — 只能 withheld 或 compensated（後者破壞 metatheory 的 commutation）
6. **Own in-memory state 不 survive reload**（forward state migration 是 future work）
7. **無 quantitative validation** — 單一 ecosystem（Koishi）+ 單一語言（TS）；agent harness 驗證明說 "compelling future direction"
8. 沒 benchmark — Proxy trap / per-effect closure / notification fan-out 的 overhead 未量化

**對 Drill 的決策輸入（§7 完整分析）：**
- **Clear win 的場景**：tool/subagent 真的需要在 runtime 頻繁 load/unload/replace + 需要 teardown 保證 + 需要 audit——這是 paper 明點的 motivating case（self-evolving agent harness）
- **Wash 的場景**：plumbing（message passing、session state 可以留在 context 外）
- **Overkill 的場景**：工具只在 boot 時載一次，之後不動——Cordis 的紀律（一切過 `ctx.effect`、inverse、disjoint provision）純成本
- **採用路徑**：先只拿 core API（`ctx.effect`/`ctx.get/set`/`ctx.use`）；每個 tool/subagent/session 當 component（inject/provide）；per-subagent 用 isolation realm；用 Thm 73 當 test oracle（quiescent state = static config）；bad inverse 用「dispose 冪等」test harness 抓

---

## 11. Honcho（memory layer 選項）

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

---

## 重要備註（命名陷阱 / 校正）

- **OpenCode canonical repo**: `anomalyco/opencode`（197k stars，opencode.ai）。注意 `sst/opencode` → `opencode-ai/opencode` 已 archived，遷移到 charmbracelet/crush。
- **deepseek-harness 名稱混淆**: 官方（`deepseek-ai/deepseek-harness`，2026-08-13 開源，75k stars，MIT，Cordis-based）vs **HenryZ838978/deepseek-harness**（社群 Python client + MCP server，處理 V4 16 quirks）vs **morlay/deepseek-harness**（PI-based 個人 agent，已廢棄）。
- **oh-my-opencode 已改名**: → `code-yeongyu/oh-my-openagent`（GitHub redirect，同一專案）。
- **Odysseus repo 遷移**: `pewdiepie-archdaemon/odysseus` → `odysseus-dev/odysseus`。`odysseusai.dev` 是第三方 guide，不是官方站。
- **NotebookLM rename**: 2026-07-16 → Gemini Notebook，同一產品。
- **PI 角色澄清**: `earendil-works/pi` = agent toolkit；`pi-ai` 是 provider abstraction 子套件；PI 本身**不做**量化（只有 OpenRouter routing filter），量化在 llama.cpp/Ollama。
- **DeepSeek-V4-Flash-0731**: HF card 顯示 304B params（含 DSpark draft module）；「284B/13B」是官方 paper 數字（arXiv 2606.19348），非 HF card。
- **n8n license**: Sustainable Use License（fair-code），embed 商用要查 LICENSE.md 門檻。
- **Overleaf license**: AGPL-3.0；embed 要四服務（realtime + updater + docstore + history），獨立性低。
- **Honcho license**: AGPL-3.0（若 Drll 要商用需注意）。
