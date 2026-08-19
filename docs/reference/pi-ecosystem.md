# PI 生態系（pi.dev 核心 + plugins）

> Track A（PI-based）的主要複用對象。調查原則：**能用就複用**。
> 深化設計見 [tracks/pi.md](../tracks/pi.md)；總覽見 [README.md](README.md)。

---

## PI (pi.dev) 核心

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

---

## Plugins（調查結果：能用就複用）

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

### 學術研究工具（Writing 直接可用）

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
| **@quintinshaw/pi-dynamic-workflows** | [github.com/QuintinShaw/pi-dynamic-workflows](https://github.com/QuintinShaw/pi-dynamic-workflows) | Fan-out subagents（**上限 16 併發 / 每 run 1000 總數**，DeepWiki 實證）：real model routing、token/cost accounting、resume、git-worktree isolation、**/deep-research** |

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
