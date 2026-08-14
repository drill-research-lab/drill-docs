# DSH — DeepSeek Harness 設計文件

> 本文為 Drill（multi-agent LLM research platform）評估 **DeepSeek Harness (`dsh`)** 作為底層基礎的單一參考文件。
> 所有論述均以 2026-08-14 之 repo 快照（commit `47f943859bef60e4160492346772ded9b24f765a`）實地驗證，
> 引用皆附 GitHub permalink。文體比照 `docs/reference.md`：繁體中文敘述 + 英文技術詞 + 表格 + markdown code block。

---

## TL;DR — 速覽表

| 面向 | 結論 |
|---|---|
| 定位 | 開源 **agent harness**（非框架、非單一 agent 產品），由 DeepSeek AI 開發；口號「Everything is a Plugin」 |
| 授權 / 釋出 | MIT；repo 建立於 **2026-08-13**；**尚無 tagged release**（`latestRelease: null`），版本 `0.1.0-rc.5`；明確標示 developer preview、將有 breaking changes |
| 底層 | **Vendored Cordis**（`cordis 4.0.0-rc.7` 等 9 個套件以 source copy 形式納入並 rename 為 `@deepseek-ai/*`）；**Koishi 在 repo 中零提及** |
| 核心哲學 | 「Everything is a plugin」：model adapter、tool registry、session log、agent loop 全是 Cordis plugin；無 privileged core；註冊皆為 reversible effect |
| 架構骨幹 | **Capability seam**（Service Definition / Service Provider / Consumer 三角色）；換 provider 即換整個產品行為 |
| Agent runtime | `Agent` interface + `ReactLoopAgent`（turn/step model）；session log 為 append-only `SessionEvent` 流，**model-visible ⟺ logged** 不變式 |
| Session | JSONL（預設，可 zstd）與 SQLite 兩種 persistence backend；`session-query` 提供 lineage / replay / resume / fork / FTS5 搜尋（後者 opt-in） |
| Tools / Skills | `ctx.tools.register(ToolDefinition)` + guarded pipeline（pre-execute/execute/post-execute waterfall）；skills 用 SKILL.md + frontmatter，`skill` tool 動態載入 |
| Subagent | `ctx.subagents` seam；in-process（spawn/fork）、ACP、codex、claude-code、dsh-sdk 五類 provider；`maxDepth` 預設 3 |
| Sandbox | `ctx.sandbox` seam：Linux bwrap → Landlock fallback、macOS Seatbelt、Windows restricted-token；fail-closed（無 backend 即拒絕執行） |
| LLM | `ctx.llm` seam；預設 `dsh-llm-deepseek`（route `deepseek-official`，V4-Flash/V4-Pro）；`dsh-llm-pi-ai` 經 `@earendil-works/pi-ai` 接任意 provider |
| Profiles / Presets | `web` / `headless` 兩 profile；bundle 分層（base → web-app/headless）＋ `cordis.patch.yml` ＋ `--patch` overlay；四組 agent preset：**standard / code / minimal / cordis**（不是 "Creator"） |
| Web UI | React + Vite；`packages/client` 下 30+ `ui-*` plugin；slot 系統 `ctx.slots.register`；presentation component 被官方標為「consumables, expected to be rewritten wholesale」 |
| 外部控制 | ACP server（Agent Client Protocol / JSON-RPC stdio）；另有 JSON-RPC SDK（TypeScript client + Python SDK），protocol 無版本協商、`0.0.1` 無相容承諾 |
| MCP | 只有 **client**（`dsh-mcp-client`，tools 以 `mcp__<server>__<name>` 註冊進 `ctx.tools`）；DSH **不**作為 MCP server 暴露 |
| 成熟度 | 核心 API「Product — stable API」；但 session format v0、sdk protocol 0.0.1、UI presentation 層可整批重寫；無 auth/multi-tenancy、無 RAG/KB、無 LaTeX、cron 僅 session-local |

---

## 1. Identity & Positioning（身分與定位）

**DeepSeek Harness（`dsh`）** 是 DeepSeek AI 開發的開源 agent harness。Repo description 即為「DeepSeek Harness: Everything is a Plugin.」（[repo view](https://github.com/deepseek-ai/deepseek-harness)）。README 開宗明義：

> "DeepSeek Harness (`dsh`) is an open-source agent harness developed by DeepSeek AI. It uses an architecture where **everything is a plugin**, and is powered by [Cordis](https://github.com/cordiverse/cordis), whose design is described in _A Programming Paradigm for Spatiotemporal Composability_."
> — [README.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/README.md#L5-L7)

**與 Cordis 的關係**：DSH **vendored** Cordis 原始碼而非走 npm 依賴，理由在 [vendor/README.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/vendor/README.md#L3-L5)：

> "They are copied into this monorepo instead of being depended on via npm, so that the harness fully owns its framework layer (auditable, patchable, pinned)."

Vendor manifest 列了 9 個套件：`cordis 4.0.0-rc.7`、`@cordisjs/plugin-loader 1.0.0-rc.5`、`include 1.0.4`、`group 1.0.0`、`timer 1.1.2`、`hmr 1.0.15`、`logger-console 1.0.0`，以及 `cosmokit 1.8.1`、`schemastery 3.18.0`，全部 rename 進 `@deepseek-ai` scope（[vendor/README.md manifest table](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/vendor/README.md#L13-L23)）。原因是每個 harness package 都宣告 `cordis` 為 peerDependency，若以上游名字發佈會 squat npm 名稱。

**與 Koishi 的關係**：repo 內 **完全沒有** "koishi" 字串（`grep -rin koishi` 全 repo 零命中）。Koishi 只是同為 Cordis 生態系的上游消費者；DSH 的文件與 code 從未提及它。若要宣稱關係，只能說「DSH 與 Koishi 共用 Cordis 框架系譜」，且這是推論而非 repo 證據。

**Maturity / Release context**：
- README 明示："DeepSeek Harness is currently in _developer preview_ and is iterating rapidly. **THERE WILL BE COMPATIBILITY-BREAKING CHANGES.**"（[README.md L9-L11](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/README.md#L9-L11)）
- GitHub API：`createdAt: 2026-08-13`、`licenseInfo: MIT`、`primaryLanguage: TypeScript`、`stargazerCount: 77035`、`forkCount: 6669`、`latestRelease: null`（尚無 tagged release）
- Root `package.json`：`@deepseek-ai/dsh-root` `0.1.0-rc.5`，`pnpm@11.7.0`、Node `^22.19.0 || >=24.0.0`（[package.json](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/package.json#L2-L10)）
- 首頁：<https://deepseek.com/harness>

## 2. Monorepo Structure（Monorepo 結構）

Layout（[AGENTS.md Repository layout](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/AGENTS.md#L9-L55)）：

| 目錄 | 內容 |
|---|---|
| `vendor/` | Vendored Cordis 原始碼（manifest + sync 程序在 `vendor/README.md`） |
| `packages/<group>/<pkg>/` | 所有 `@deepseek-ai/dsh-<pkg>` workspace |
| `apps/` | `cli`（`dsh` CLI）、`web`（Vite frontend `@deepseek-ai/dsh-web-frontend`） |
| `python/` | Python SDK（`sdk` client + `sdk-runtime` 執行環境） |
| `native/` | `@deepseek-ai/node-addon-landlock-run`（Landlock C 工具）source of record |
| `examples/` | 可執行的 `cordis.yml` leaves（acp-agent / headless-agent / jsonrpc-agent / mcp-memory / web-cordis / web-schedule） |
| `website/` | VitePress 文件站 |
| `docs/` | architecture、catalogs、cookbook、subsystems reference |
| `.agents/` | Agent workflows 與 Agent Notes |

命名慣例：npm scope `@deepseek-ai/dsh-*`；group 目錄是純容器（無自己的 `package.json`）；`tsconfig.base.json` 以 wildcard `@deepseek-ai/dsh-*` 對映（[packages/README.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/README.md#L7-L9)）。

Package groups（[packages/README.md Hierarchy table](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/README.md#L11-L59)，節錄）：

| Group | Role | Release expectation |
|---|---|---|
| `core/` | Product API spine：sessions、prompts、tools、agent services、concrete loop | Product — stable API |
| `api/` + `typert/` | Remote BFF assembly、Typert RPC gateway（type graph 生成/載入/runtime registry） | Product — stable API |
| `llm/` | LLM capability family：抽象 service + provider adapters（`llm`、`llm-deepseek`、`llm-pi-ai`、`llm-retry`、`token-meter`） | Product — stable API |
| `shell/` `subprocess/` `terminal/` `fs/` `lsp/` `web/` | 各執行世界 capability family（Service Definition + provider + tool Consumer） | Product — stable API |
| `code-runtime/` | Code 執行 seam + worker-thread provider + Code Mode Consumer | Product — stable API |
| `sandbox/` | Process-confinement seam；bwrap/Landlock/Seatbelt backends | Product — stable API |
| `skill/` | Skill capability family：provider registry + local impl + catalog/loader tool | Product — stable API |
| `subagent/` | Subagent capability family：provider-registry contract + delegation tools | Product — stable API |
| `session/` + `session-query/` | Durable session data plane（persistence seam + JSONL/SQLite）、retrieval family（FTS5、lineage） | Product — stable API |
| `sdk/` `acp/` | Out-of-process 控制：JSON-RPC protocol/client/server、ACP server | Product — stable API |
| `client/` + `host/` | Web GUI 兩半：browser half（shell、wire、slots、`ui-*`）＋ host half（API gateway、HTTP server） | Product — stable API |
| `bundle/` | Installable `dsh --profile` patch layers（`base`、`web-app`、`headless`） | Product — stable API |
| `e2b/` | E2B remote runtime | **POC** |
| `examples/` `test-support/` `util/` | Demo bundles、測試基建、零依賴工具 | Support |

**「Everything is a plugin」在此具體為**（[architecture.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/architecture.md#L9-L13)）：

> "Every part of the product is a plugin, including the model adapter, the tool registry, the session log, and the agent loop itself, so every part is replaceable from configuration. There is no privileged core to patch: you extend dsh by mounting a plugin beside the others, and registrations are effects that unwind when their plugin unloads."

## 3. Cordis Kernel Integration（Cordis 核心整合）

Cordis 五個核心概念（[docs/cordis-primer.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/cordis-primer.md)「Cordis In Five Ideas」）：plugin 是 function 或 Service subclass；context 是 service 倉庫（`ctx.<key>`）；用 `inject` 宣告依賴；typed events（`emit`/`waterfall`/`parallel`/`serial` 四種 dispatch mode）；註冊是 reversible effects（`ctx.effect()`/`ctx.on()` 回傳 disposer，unload 時自動 unwound）。

**Typed events 用 declaration merging**（[AGENTS.md conventions](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/AGENTS.md#L104)）：`SessionEventMap` 是 merge-extensible map，plugin 新增 session event type 不需改核心套件；`SessionEventMap` member 預設 required-on-read（不認得該 event 的 build 會拒絕 log，除非帶 `ignorable: true`）。

**Capability seam**：三角色（[docs/glossary.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/glossary.md)）：

> "a **Service Definition** (the Cordis `Service` that owns its `ctx.<key>` and vocabulary types…), one or more **Service Providers**, and one or more **Consumers** that inject the service."

**真實範例 — web seam**（完整三角色）：

**Service Definition** — `@deepseek-ai/dsh-web`（[packages/web/web/src/index.ts](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/web/web/src/index.ts#L1-L66)）：

```ts
/** Service Definition for the web access capability seam (`ctx.web`)... */
export class WebRuntime extends Service {
  static Config: z<WebRuntimeConfig> = z.object({ ... })
  // 執行時 provider selection：configured id、auto-select、AMBIGUOUS/UNAVAILABLE 錯誤語彙
  registerSearchProvider(id: string, provider: WebSearchProvider): () => void
  registerFetchProvider(id: string, provider: WebFetchProvider): () => void
}
declare module '@deepseek-ai/cordis' {
  interface Context { web: WebRuntime }
}
```

**Service Provider** — `@deepseek-ai/dsh-web-search-deepseek`（[index.ts](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/web/web-search-deepseek/src/index.ts#L1-L20)）：

```ts
export const name = 'web-search-deepseek'
export const inject = ['web']
// apply() 內：ctx.web.registerSearchProvider(DEEPSEEK_PROVIDER_ID, new DeepSeekSearchProvider(...))
```

同族 provider 還有 `web-search-exa`、`web-search-perplexity`、`web-fetch-http`。

**Consumer** — `@deepseek-ai/dsh-tool-web`（[index.ts L2](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/web/tool-web/src/index.ts#L2)、L24）：

```ts
export const inject = ['tools', 'web', 'systemPrompt']
// 註冊 model-facing `web_search` / `web_fetch` tool schema，執行時呼叫 ctx.web
```

關鍵特性：consumer 不 import 具體 provider；provider 換掉（本地 → 遠端 sandbox）時 bash、PTY、LSP 等同一執行世界的 consumer 全部一起搬移（[architecture.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/architecture.md#L98-L102)）。

## 4. Agent Runtime（Agent 執行期）

**`Agent` interface**（`packages/core/agent`，[types.ts](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/core/agent/src/types.ts)）：`followup()`（普通訊息入 inbox、立刻 wake driver）、`steer()`（當前 turn 下一 step 的 steering input）、`inject()`（model-facing context，不 wake）、`cancel()`。Live registry 為 `ctx.agents`。

**Turn/Step model**（[docs/architecture.md Turn flow](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/architecture.md#L63-L82)）：

> "A **step** is one model request plus the tools it calls. A **turn** is zero or more steps: it opens before its first input is claimed and closes once nothing is owed."

```
turn/start
  claim next-step input plus one queued message
  assemble prompt sections + tool schemas
  -> agent/pre-step                   reject | enter(messages)
     step/start
     append entered messages as user/message
     derive model history from the log
     agent/request -> llm/stream -> assistant/chunk* -> assistant/message
     tool/call* -> tools/pre-execute -> tools/execute -> tools/post-execute -> tool/result*
     step/end
     tools owe another request, or next-step input arrived -> claim -> next step
  -> agent/turn-stopping
turn/end
```

- `turn/*`、`step/*`、`user/message`、`assistant/*`、`tool/*` 是 **durable session events**；`agent/*` 是 live extension points。
- Waterfall events（listener 必須 call `next()` 否則 short-circuit）：`agent/pre-step`、`agent/request`、`llm/stream`、三個 `tools/*`；`agent/turn-stopping` 是 serial、無 `next()`（[architecture.md L84](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/architecture.md#L84)）。
- 實作 `ReactLoopAgent` 在 `packages/core/agent-loop`（internal，plugin 透過 `ctx.agents` 互動）。

**System-prompt assembly**：`ctx.systemPrompt`（`core/system-prompt`）彙整註冊的 prompt sections + tool schemas；`system-prompt/assemble` 是 waterfall；`renderPrompt(assembly)` 做 `{{variable}}` interpolation。persona 是 `deployment:persona` section（[packages/preset/persona/src/index.ts](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/preset/persona/src/index.ts)）。

**Session events（append-only log）**：`core/session`（`ctx.sessions`）。每個 `SessionEvent` 有 monotonic `seq` + Unix `time` envelope。`SessionEventMap` declaration merging 定義 event vocabulary。

**`deriveMessages()`**：從 log 投影 model 看的 `Message[]`：`user/message` → user message；`assistant/message` → assistant message（raw `assistant/chunk` 被跳過，因組裝後的 message 才 authoritative）；`tool/result` → user message 含 tool-result block；其餘 turn/step 是 structural、不投影。

**Session-query**（`packages/session-query/`，[packages/README.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/README.md#L46)）：
- `session-query` — `SessionQueryEngine`：`searchSessions()`/`searchEvents()`、`traceSession()`（lineage）、`listSessions()`/`filterSessions()` 等
- `session-query-sqlite` — SQLite FTS5 全文搜尋實作（[index.ts L2](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/session-query/session-query-sqlite/src/index.ts#L2)、[query.ts L29-L52](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/session-query/session-query-sqlite/src/query.ts#L29-L52)）；**opt-in**（base bundle 設 `openAt: never`，見 §14）
- `tool-session-query` — model-facing 查詢 tool
- `session-log-export` — 匯出

**Replay / Resume / Fork**：replay = 以既有 event log 為 seed 開新 session（`Session.create()`）；resume = `ctx.agents.resume({ resumeSessionId })`（`ctx.sessionPersistence.prepare()` 取回 unpublished session 再 publish，續用 turn numbering、append `session/end-seed`）；fork = `ctx.sessions.fork(source, boundary?, childSessionId?)`，child header 記錄 `parentSession` + `seedLength`，boundary 必須結束在 open turn 之外。

**Persistence backends**：`session-persistence`（seam）＋ `session-persistence-jsonl`（預設；append-only、zstd 壓縮、chunk-row packing，[format.ts](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/session/session-persistence-jsonl/src/format.ts#L17-L25)）＋ `session-persistence-sqlite`。

**Session format 版本**：`SESSION_FORMAT_VERSION = 0`（[packages/core/session/src/types.ts L56](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/core/session/src/types.ts#L56)，header 不符即 throw，[index.ts L101-L102](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/core/session/src/index.ts#L101-L102)）。

## 5. Tools & Skills（工具與技能）

**Tool registration** — `ctx.tools.register(definition: ToolDefinition)`（[core/tools/src/index.ts L1037](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/core/tools/src/index.ts#L1037)）回傳 disposer。`ToolDefinition` 含 name、description、JSON Schema params、`execute`、`finalizeContent`。Agent-local 註冊會 shadow 同名 global tool（scope 機制，見 [docs/glossary.md agent-scope](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/glossary.md)）。重複註冊會 throw（L727-L728）。

**Guarded execution pipeline**（[docs/tool-execution-pipeline.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/tool-execution-pipeline.md)）：`tools/pre-execute`（allow/deny/ask waterfall）→ monotonic guards（`ctx.tools.guard()`，不可繞過）→ `tools/execute`（around-dispatch：timeout/retry/metrics）→ dispatch → `tools/post-execute`（enrich）→ `finalizeContent` → `tools/result`（observe-only terminal event）。

**Tool presentation**：`native` / `code` / `both` 三種 mode。Code Mode 下只露 `run_code` transport + 依 `ctx.codeRuntime.language`（TypeScript/Python）產生的 SDK `.d.ts` 進 system prompt（[core/tools/src/index.ts L866-L888](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/core/tools/src/index.ts#L866-L888)）。

**Tool catalog**：`docs/tool-catalog.md` 為 generated（boot 所有 plugin 後 call `ctx.tools.schemas()`）；CI 以 `gen-tool-catalog --check` gate（[package.json scripts](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/package.json#L112-L113)）。

**Skills**（`packages/skill/`）：
- `skill` — `SkillRegistry`（`ctx.skills`）：`registerProvider()` 合併多來源 skills、處理衝突
- `skill-filesystem` — 本機 provider：掃 `.dsh` / `.agents` / custom / user roots（rank order：project `.dsh` → project `.agents` → customSkillDirs → user `.dsh` → user `.agents`）；解析 `<name>/SKILL.md` 或 flat `<name>.md`
- `skill-badge`、`tool-skill` — `dsh-tool-skill` 在每個 `agent/pre-step` 用 `ctx.skills.snapshot()` 注入 `<system-reminder>` catalog（有變才重發）；`skill(name=...)` tool 載入完整 instructions
- SKILL.md 支援 YAML frontmatter：`name`、`description`、`whenToUse`、`metadata`、`disable-model-invocation`、`user-invocable`（name + description 必填）

## 6. Subagents & Multi-Agent（子代理與多代理）

`ctx.subagents` seam（`packages/subagent/subagent/`）：`SubagentRuntime` 管理 model tool call → child agent 執行。Provider 以唯一 name 註冊，宣告 `outputSchema`、`depthLimit`、`toolFilter`、`persona` 等 capabilities。

**Provider implementations**（[packages/subagent/](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/subagent/)）：

| Provider package | 類型 |
|---|---|
| `subagent-spawn-in-process` | 同 process 全新 child context |
| `subagent-fork-in-process`（+ `subagent-in-process-driver` 共用 driver） | 同 process child，繼承 parent 已完成 turns |
| `subagent-acp` | 經 ACP JSON-RPC stdio spawn 外部 process（repo 內 primary ACP client） |
| `subagent-codex` | 真實 Codex app-server child |
| `subagent-claude-code` | 真實 Claude Code child（Claude Agent SDK） |
| `subagent-dsh-sdk` | 遠端/特製 runtime 以 SDK 註冊為 provider |

**Delegation depth guard**（[subagent/src/child-agent.ts L48-L54](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/subagent/subagent/src/child-agent.ts#L48-L54)）：

```ts
export function resolveChildDepth(parent: Agent, maxDepth: number | undefined): number {
  const childDepth = delegationDepthOf(parent) + 1
  ...
  if (maxDepth !== undefined && childDepth > maxDepth) {
    throw new SubagentDepthError(childDepth, maxDepth)
  }
```

`maxDepth` 預設 **3**（[tool-subagent/src/index.ts L98](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/subagent/tool-subagent/src/index.ts#L98)）；`delegationDepth` 持久化於 `SessionHeader` 且 monotone（只增不減）。`maxDepth: 'provider-managed'` 可把遞迴預算交給 provider（L285-L288 會檢查 provider 是否有 `depthLimit` capability）。

**Child-to-parent communication**：continuable subagent 用 `reportFrom(child, content, { delivery, signal })`，`delivery: 'quiet'`（inject context）或 `'wakeup'`（觸發 parent 新 turn）；one-shot subagent 則在 `SubagentRun.result.output` 拿 child 最後的 assistant 內容。相關 tools：`tool-subagent`（delegation）、`tool-subagent-control`（child messaging/listing）、`tool-subagent-report`（child→parent report channel）。

## 7. Sandbox & Code Execution（沙箱與程式執行）

**`ctx.sandbox` seam**（[packages/sandbox/sandbox/README.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/sandbox/sandbox/README.md)）：
- Vocabulary：`SandboxMode`（`read-only` / `workspace-write` / `danger-full-access`，僅 file effects）、`SandboxEnforcement`（`full`/`partial`）、fail-closed `SANDBOX_UNAVAILABLE`
- 合約一句話：`ctx.sandbox.confine(argv, policy)` 回傳**替換**的 wrapped argv；無可用 backend 時 throw，**絕不**裸跑
- **Backends**（`dsh-sandbox-local`，[src/index.ts L2](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/sandbox/sandbox-local/src/index.ts#L2)）：Linux 先 probe `bwrap`（`--ro-bind --dev --proc --die-with-parent`，[L69](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/sandbox/sandbox-local/src/index.ts#L69)），fallback 到 Landlock launcher（`@deepseek-ai/node-addon-landlock-run`，native C11 + Landlock UAPI）；macOS `sandbox-exec`/Seatbelt（L85-L86）；Windows `WRITE_RESTRICTED` tokens（`dsh-sandbox-windows-acl`）
- **Known limitation**：same-world confinement only — container/microVM/remote executor 不屬於此 seam，要換整個 capability seam 的 provider（`ctx.shell`/`ctx.fs`）

**`ctx.codeRuntime`**（`packages/code-runtime/`）：執行 model 寫的程式。`dsh-code-runtime-worker-thread` provider（[README](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/code-runtime/README.md)）：每次 `run()` → Node `stripTypeScriptTypes` → spawn 全新 Worker（空環境、可設 resource limits）→ 以 `AsyncFunction` body 執行 → 三個獨立 budget（`computeMs`/`maxWallMs`/`maxOutputBytes`）。**不是硬安全邊界** — model code 的 authority 與 bash tool 相當。

**Code Mode**：`tools` config `mode: code` 時，registry 只露 `run_code` tool + generated SDK；`run_code(code, description)` 提交 TypeScript 程式。headless patch 用 `mode: !!js process.env.DSH_TOOLS_MODE` 維持與 Web 相同 opt-in（[headless/cordis.patch.yml](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/bundle/headless/cordis.patch.yml)）。

**`ctx.fs` capability family**（`packages/fs/`）：`fs`（seam）＋ `fs-local`、`fs-sandbox`（per-call mode fence：`read-only` 全拒、`workspace-write` 限 writable roots、`danger-full-access` 直通）、`tool-fs`、`tool-fs-search`、`tool-str-replace-editor`、`fs-observation-policy`。**限制**：text-only mutations by contract、僅 12 個 primitives、無 delete/rename/move/copy/watch（[fs README](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/fs/fs/README.md)）。

## 8. LLM Providers（LLM 供應商）

**`ctx.llm` seam**（`packages/llm/llm`）：`LlmRuntime` 是 central registry + dispatch hub。統一 `StreamChunk` vocabulary（`block-start`/`block-end`、`text-delta`、`reasoning-delta`、`usage`、`finish`），`BlockAssembler` 重組完整 message。`LlmAdapter` abstract class 是 provider backend 必實作的介面（`providerInfo`、`listModels`、`stream`）；`ctx.llm.registerAdapter(providers, adapter)` 是 atomic、fiber-scoped。

**`dsh-llm-deepseek`**（[README](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/llm/llm-deepseek/README.md)）：route `deepseek-official`；raw `fetch` + 自寫 SSE parser；`reasoning_effort`/`thinking` 支援；idle watchdog；credential 每 call 解析。**預設 models**：`deepseek-v4-flash` + `deepseek-v4-pro`，各 1,000,000-token context window；`maxTokens` 預設 256,000（L38-L42）。`settings` 的 `models` list 整批取代 composition list；`tool_choice` 未對映（L111-L112）。

**`dsh-llm-pi-ai`**（PI interop，[README](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/llm/llm-pi-ai/README.md)）：backed by `@earendil-works/pi-ai` 的 generic multi-provider adapter。一個 plugin instance 持 provider profile dict（keyed by route）；每個 request 以 `GenerateOptions.provider` 選 profile、以 route 的 catalog 解析 model。pi-ai 已內建 provider（OpenAI、Anthropic…）繼承 endpoint/protocol/catalog 為 default，可逐 field override；未內建 route 直接宣告，故「新 provider = 設定而非改 code」。

**`llm-retry`**：兩層 retry — provider-level（`ctx.llm.providerRetryPolicy(provider)`）與 agent-level（plugin 攔 `llm/stream` waterfall + 聽 `agent/request-error`）。adapter 把 HTTP status/provider errors 正規化成穩定 `LlmError` codes。

**`token-meter`**：replay-aware token 量測；counts 互斥（`inputTokens` / `cacheReadTokens` / `cacheWriteTokens`）。

**Model routing/fallbacks**：`ctx.llm.prepareCall(config, signal)` 解析 model metadata（context window、reasoning efforts）並把 adapter registration + retry policy 凍結成 one-shot executable；`resolveModelInfo` 查 exact model identity/capabilities。

## 9. Profiles, Bundles, Modes（設定組成）

**Layering**（[docs/architecture.md Profiles and bundles](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/architecture.md#L15-L37)）：

```
每個 bundle（依 profile 列出的順序）→ profile 的 cordis.patch.yml → home-level cordis.patch.yml → 任何 --patch overlay
patch 以 row id 為目標，整包取代該 row 的 config，或 insert 新 rows
```

- **profile** = Harness home 底下具名 composition（`$DSH_HOME/profiles/<name>`）；`web` 與 `headless` 以 template 隨附；`dsh --profile web --dump-config` 可印出真正 boot 的 tree
- **bundle** = 分發格式：`package.json` 的 `dsh.bundle` field 指向其 `cordis.patch.yml`；`dsh-base` 是每個 profile 第一層（model adapters、tools、persistence、sandbox、approval policy、settings、credentials、telemetry）；`dsh-web-app` 加 browser application；`dsh-headless` 加 one-shot runner（無 server）（[architecture.md L25](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/architecture.md#L25)）

`dsh-base` patch 內容例（[base/cordis.patch.yml](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/bundle/base/cordis.patch.yml)）：`timer`、`hmr`、`llm`、`session`、`typert-*`、`session-title-*`、`agent`、`agent-default-model`（provider `deepseek-official` / model `deepseek-v4-flash`）、`jobs-local`、`llm-retry`、`settings-file`。

**Agent presets**（`packages/preset/` + 隨附 presets 在 [apps/cli/config/agent-presets/](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/apps/cli/config/agent-presets/)）— **實際四個是 `standard` / `minimal` / `code` / `cordis`**（任務描述中的 "Creator" 不存在；第四個叫 `cordis`，是 self-referential 的 meta-agent）：

| Preset | 內容（agent.cordis.yml） |
|---|---|
| `standard` | 完整 coding agent：persona + instructions + `tool-bash`/`tool-pwsh`/`tool-fs`/`tool-fs-search`/`tool-jobs`/`tool-skill`/`tool-goal`/plan/compaction/delegation（含 codex、claude-code）/`tool-ask-user`/`tool-todo`/`tool-web`（`fetch: false`） |
| `minimal` | 固定 prompt 的雙 tool agent：persona（`complete: true` + `includeRuntimeContext: false`）+ 僅 persistent `bash` + `str_replace_editor`；無 compaction |
| `code` | `standard` 全部 + `tool-presentation` row → Code Mode（model 寫 TS 程式打 generated SDK，五個 round trip 變一個） |
| `cordis` | `standard` 全部 + self-referential Cordis toolset（agent 可讀寫自己所在的 runtime、auth 另一個 agent）；TRUST 標示「等同 shell access」 |

Preset 機制（[agent-presets/README.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/preset/agent-presets/README.md)）：preset = 一個目錄含 `agent.cordis.yml`；roster 每 process 只 mount 一次（standing scope），各 session 以 scope parentage 加入；service row 必須在 `isolate` realm 內否則跨 session 撞名；authoring 只允許 copy（`ctx.agentPresets.copy`），id 限制 `[a-z0-9][a-z0-9-]*`。

## 10. Web UI（前端）

**Stack**：React + Vite；`apps/web`（`@deepseek-ai/dsh-web-frontend`）是 Vite entry，疊在 `@deepseek-ai/dsh-client-web` shell 之上。

**`packages/client/` 佈局**（[client/README.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/client/README.md)）：

| 層 | Packages |
|---|---|
| Shell boot | `web`、`modules` |
| Wire/connection | `connection`（HTTP POST unary + WebSocket `events.mux`/`events.host` streaming） |
| Object services | `runtime`（React-free snapshot store engine：session/workspace/UI composition） |
| React glue | `web-react` |
| Slot system | `ui-slots` |
| Primitives | `ui-primitives`、`ui-theme`、`locale`、`schema-form`、`hmr` |
| Feature plugins | `ui-layout`、`ui-sidebar`、`ui-workspace`、`ui-conversation`、`ui-tool`、`ui-trajectory`、`ui-subagent`、`ui-jobs`、`ui-plan`、`ui-goal`、`ui-settings*`（general/models/plugins/plugin-inventory）、`ui-agent-preset`、`ui-skill`、`ui-commands`、`ui-input-trigger`、`ui-attachment`、`ui-message-feedback`、`ui-user-questions`、`ui-model-selection`、`ui-permission-presets`、`ui-workflow-run`、`ui-directory-picker-*`、`ui-deliverables` 等 30+ |

**Slot 系統**（`ctx.slots.register`）：plugin 以 `ctx.slots.register({ name, children?, store?, inject? }, Component)` 註冊 UI；`SlotMap` declaration merging 是 slot 型別 authority（[.agents/notes 2026-07-22-slot-type-chain-implementation](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/.agents/notes/implemented/architecture/2026-07-22-slot-type-chain-implementation.md)）。加自訂頁面 = 註冊 slot + keyed renderer（architecture.md 的「Add a Web Client Chat node → register a ConversationNodeDefinition + keyed renderer」，[L122](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/architecture.md#L122)）。

**Stability notes**（[packages/client/AGENTS.md L48](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/client/AGENTS.md#L48)）：

> "**Presentation components** (plugin packages' `src/client/`, pure props): consumables, expected to be rewritten wholesale. Business logic must not leak into them; everything arrives through the four props shares."

即：資料物件層（`runtime`/snapshot store）與 slot 機制相對穩定，但**所有視覺 component 被官方視為消耗品、可整批重寫** — 不要在其上建不可拋棄的 UI 邏輯。

**Host 半**（`packages/host/`）：`apiproxy`（transport-independent API gateway + Typert type-safe RPC，四象限 message model + Zod validation）、`webserver`（`node:http` plugin，`ctx.webServer`，static serving + `/api/*` forwarding + WebSocket upgrade）。

## 11. ACP & External Control（外部控制）

**ACP server**（`packages/acp/acp`，[README](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/acp/acp/README.md)）：Agent Client Protocol over JSON-RPC stdio；**stdout 完全保留給 protocol frames**（所有 logging 抑制）。對外方法：

| Method | 行為 |
|---|---|
| `initialize` | 版本協商；只廣告 baseline prompts（無 image/audio/embedded-context、無 MCP/fs/terminal capability） |
| `authenticate` | No-op（不廣告 auth methods） |
| `session/new` | 以絕對 `cwd` 建 fresh agent（`additionalDirectories`/`mcpServers` 非空即 reject） |
| `session/prompt` | 送 text prompt、等整個 agent idle；正常結束 `end_turn`、取消/棄置 `cancelled` |
| `session/cancel` | 只 cancel 被指名的 agent |
| `session/update` | 對 committed `assistant/message` 的每個 non-empty text block 發 `agent_message_chunk` |
| `session/request_permission` | bridge-owned approval 的 one-shot allow/reject |

**外部 orchestrator 的互動模式**：spawn 一個 `dsh` ACP 實例（`pnpm run demo:acp` / [examples/acp-agent/cordis.yml](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/examples/acp-agent/cordis.yml)），以 `session/new` 開 session、`session/prompt` 送任務、`session/update` 收 committed text。工具**不是**由 orchestrator inject — 由 DSH 自身 cordis.yml 決定；ACP 是刻意窄化的 automation 面，不暴露 replay/commands/modes/titles（README 明言）。

**SDKs**：
- TypeScript：`packages/sdk/client`（`DeepSeekHarness` high-level class 管 subprocess lifecycle + `HarnessClient` 低階 JSON-RPC）
- Python：`python/sdk`（`deepseek-harness-sdk`，mirror TS 設計）
- **Wire protocol** `packages/sdk/protocol`（version `0.1.0-rc.5`；[Known Limitations](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/sdk/protocol/README.md)）：**無 protocol-version negotiation**、`serverInfo.version` 為 `0.0.1` 且 client 不驗證、**pre-release、無相容承諾**；無 cancel/close methods（client 直接關 process）；server→client requests 是 dead capability

## 12. Plugin Authoring（外掛撰寫）

**Minimal plugin**（[docs/cordis-primer.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/cordis-primer.md) 與 cookbook）：

```ts
import { Service, type Context } from '@deepseek-ai/cordis'

// 1) function plugin + inject 宣告
export const name = 'my-tool-plugin'
export const inject = ['tools']          // ctx.tools 就緒後才跑 apply
export function apply(ctx: Context) {
  ctx.tools.register({ name: 'my_tool', description: '...', execute: async () => 'hi' })
}

// 2) 或貢獻 service：extends Service + declaration merging
declare module '@deepseek-ai/cordis' {
  interface Context { greeter: GreeterService }
}
export class GreeterService extends Service {
  constructor(ctx: Context) { super(ctx, 'greeter') }
  greet(who: string) { return `Hello, ${who}!` }
}
```

- `ctx.effect()` — 註冊需 cleanup 的資源，回傳 disposer（unload 時自動跑）
- `ctx.on(event, listener)` — 事件監聽（unload 自動移除）；waterfall listener 必須 `next()` 否則短路（AGENTS.md：「Waterfall listeners MUST call `next()` to delegate」[L106](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/AGENTS.md#L106)）
- 「Registrations are effects」是 repo 硬性慣例（AGENTS.md L102）

**Extension cookbook**（[docs/cookbook/](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/docs/cookbook/)）：`extension-cookbook.md`（feature→mechanism 對照表）、`adding-a-tool.md`、`adding-a-package.md`、`adding-an-llm-adapter.md`、`adding-a-conversation-node.md`、`adding-a-vendored-package.md`。

**Examples**：`examples/`（runnable `cordis.yml` leaves）+ `packages/examples/`（demo bundles：`acp-demo`、`jsonrpc-demo`、`agent-spine-demo`）。最實用的起步點：`examples/jsonrpc-agent/`（minimal.py + minimal.cordis.yml + sdk.snapshot.ts）。

## 13. MCP Support（MCP 支援）

**只有 client**。`packages/mcp/mcp-client`（[packages/mcp/README.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/mcp/README.md)）：單一套件、單一 MCP server 需一個 plugin instance；支援 `stdio`（spawn child process）與 `streamable-http` 兩種 transport。

**註冊流程**：`tools/list` fetch phase（先建新 generation 的 `ToolDefinition`、不動 registry）→ swap phase（dispose 舊 tools、`ctx.tools.register()` 新 tools）。公開名稱採 server-qualified：`mcp__<serverName>__<rawName>`（防命名碰撞）。

**DSH 是否作為 MCP server**：**否**。repo 內只有 client bridge。

**限制**（mcp-client README）：只 bridge **tools**（resources/prompts deferred）；無 connection/discovery timeout（繼承 MCP SDK 預設 60s）；image/audio/resource payload 在 model context 變 placeholder；超出 harness subset 的 output schema 的 `structuredContent` fallback 成 `JsonValue`。

## 14. Honest Maturity Assessment（誠實的成熟度評估）

**項目自己怎麼說**：
- README：「developer preview…THERE WILL BE COMPATIBILITY-BREAKING CHANGES」
- AGENTS.md 開頭「**Pre-release stance: foundation over blast radius**」："With no external consumers, prefer the correct foundation over compatibility shims: rename or repackage freely… Backends reject old on-disk formats. SQLite uses monotonic `SCHEMA_VERSION`; `dsh-session` keeps `SESSION_FORMAT_VERSION` at `0` with no compatibility promise." +「Remove this section at the first tagged release.」（[AGENTS.md L5-L7](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/AGENTS.md#L5-L7)）
- SDK protocol：`0.0.1` unvalidated、no compatibility promise

**穩定 vs 可能 churn**：

| 層 | 狀態 |
|---|---|
| Capability seam 架構、`core/` 產品 API（session/prompt/tools/agent） | 官方標「Product — stable API」；架構慣例（effects、waterfall、scoped registration）被 gates 強制 |
| On-disk session format | `SESSION_FORMAT_VERSION = 0`，**無相容承諾** |
| SDK wire protocol | `0.0.1`、無協商、無相容承諾；API 常隨 rc 版本變 |
| Web client presentation components | 官方明言「consumables, expected to be rewritten wholesale」 |
| `e2b/` | 官方標 POC（ephemeral state、無 host sync、無部署平台） |
| `web-fetch-http` | SSRF/private-network 保護 deferred；README 警告不可在能觸及內部網路的部署啟用 |
| `fs` | 12 primitives、text-only、無 delete/rename/copy/watch |
| `session-query-sqlite` | FTS5 全文搜尋 **opt-in**（base 設 `openAt: never`，[base/cordis.patch.yml L109-L121](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/bundle/base/cordis.patch.yml#L109-L121)） |
| Telemetry | 掛載但預設關閉（`DSH_TELEMETRY_MODE` opt-in） |

**缺失**（repo 內無證據支援的）：**auth/multi-tenancy**（只有 anonymous identity，無使用者/租戶模型）、**RAG/KB**（無；只有 skills + session-query）、**LaTeX**（無任何提及）、**cron** 僅 session-local（`packages/schedule`：`after_seconds`/`at`/`every_seconds`≥5min 的 Session-scoped reminders，[schedule/README.md](https://github.com/deepseek-ai/deepseek-harness/blob/47f943859bef60e4160492346772ded9b24f765a/packages/schedule/schedule/README.md)）。

## 15. Capability Status（「開箱即用」vs「需自填的 seam」）

| Capability | 狀態 | 證據 |
|---|---|---|
| LLM（DeepSeek） | ✅ ships working | base patch `agent-default-model` provider `deepseek-official` model `deepseek-v4-flash` |
| LLM（其他 provider） | ⚠️ seam，`dsh-llm-pi-ai` 預設 dormant | 需 settings 供 provider profiles（pi-ai README） |
| Session persistence | ✅ ships working（JSONL default；SQLite 選配） | base patch + persistence-catalog |
| Session 全文搜尋 | ⚠️ 需 opt-in | `openAt: never` → 改 `first-search` |
| Tools registry + guarded pipeline | ✅ ships working | `core/tools` + guard plugin |
| Skills | ✅ ships working（filesystem provider） | standard preset `skill-filesystem` + `tool-skill` |
| Subagents（in-process） | ✅ ships working | `spawn-in-process`/`fork-in-process` |
| Subagents（codex / claude-code） | ⚠️ 需外部 SDK/授權 | standard preset 有 tool rows，需真實 CLI |
| Subagents（遠端/特製 runtime） | ⚠️ seam | `subagent-dsh-sdk` 供註冊 |
| Sandbox | ✅ ships working（bwrap/Landlock/Seatbelt） | `dsh-sandbox-local`；**但**無 backend 即 fail-closed — 部署必須自備 bwrap 或支援 Landlock 的 kernel |
| Remote execution world | ⚠️ 缺省 | `e2b/` 是 POC；container/microVM 需自行實作 `ctx.shell`/`ctx.fs` provider |
| Code runtime（Code Mode） | ✅ ships working（worker-thread） | headless/web patch 皆掛 `code-runtime` |
| Web UI | ✅ ships working（web profile） | `dsh-web-app` bundle；presentation 層可重寫 |
| ACP 外部控制 | ✅ ships working | `packages/acp` + `examples/acp-agent` |
| JSON-RPC SDK | ✅ ships working（TS + Python） | `packages/sdk` + `python/sdk`；protocol 無相容承諾 |
| MCP | ✅ client ships working；**無 server 面** | `packages/mcp/mcp-client` |
| Auth / multi-tenancy | ❌ 不存在 | 全 repo 無證據 |
| RAG / KB | ❌ 不存在（skills/session-query 是最接近的替代） | — |
| LaTeX / 文件渲染 | ❌ 不存在 | — |
| Cron | ⚠️ 僅 session-local schedule | `packages/schedule` |

---

## 設計決策輸入（Drill 的 implication）

以下只陳述事實與取捨，不下 build/fork/reference 結論：

**A. 用 DSH 的價值**
1. **capability seam 是成熟的正向設計**：換 LLM provider、換 sandbox、換 subagent backend 都是換 provider row（`cordis.patch.yml`），Drill 的 research agents 可以共享同一套 execution-world 抽象而各自擁有不同 sandbox/policy。
2. **session log 是完整的 event-sourced 基礎**：append-only `SessionEvent` + lineage/fork/resume/replay 對 research platform 的「實驗可重播、可回溯、可分支」需求幾乎是量身訂做；「model-visible ⟺ logged」不變式保證可稽核。
3. **subagent 深度保護與多 provider 已實作**：`maxDepth` 預設 3、`reportFrom` child→parent channel、in-process/ACP/SDK 五類 provider — Drill 的 team-of-agents 可直接在 `ctx.subagents` 之上架構。
4. **preset + bundle 分層**讓「研究模式」與「一般模式」能以 `cordis.patch.yml` 疊層切換，不用 fork。
5. **web slot 系統 + React**：加自訂頁面（研究儀表板、實驗檢視）有明確 extension point（slot registration / ConversationNodeDefinition），且 UI 可整批重寫而資料層不動。

**B. 使用 DSH 的風險與代價**
6. **全部無相容承諾**：`SESSION_FORMAT_VERSION=0`、SDK protocol 0.0.1、developer preview、無 tagged release。任何建在其上的持久化資料或對外 API 都要自己加版本帶，並預期上游 rename/repackage。
7. **Web UI 是消耗品**：官方明言 presentation components 可整批重寫 — Drill 若深度客製 UI，等於承接這批重寫成本；business logic 必須只放在 slot store/runtime 層。
8. **缺 Drill 需要的產品面**：無 auth/multi-tenancy（多人研究平台要自建 identity/授權層）、無 RAG/KB（retrieval 要自建或接 MCP/session-query）、無 LaTeX、無排程（僅 session-local reminders）、無硬體沙箱（bwrap/Landlock 是 same-world confinement，遠端隔離要自己實作 `ctx.shell`/`ctx.fs` provider 或使用 E2B POC）。
9. **環境相依**：sandbox 依賴 host 有 bwrap 或 Landlock kernel / macOS sandbox-exec；`dsh-llm-deepseek` 為預設但 Drill 若要 DeepSeek 以外模型需自行組 `llm-pi-ai` profile。
10. **repo 大、gates 多**：100+ verify scripts、per-file 100% coverage gate、雙語文件 gate — fork 後維護成本高；但作為 reference 或 plugin 生態參與（`dsh-plugin` topic）成本低。

**C. 可複用的具體零件（不 fork 也能參考）**
11. **session-query 的 lineage/trace 模型**（`traceSession` 沿 `parentSession` 重建 ancestor tree）是 research platform「實驗家族樹」的現成 schema 靈感。
12. **sandbox `confine(argv, policy)` 的 per-call policy 設計**（policy 跟著 call 走而非 provider）適合 research 平台不同 experiment 不同權限的場景。
13. **`agent/pre-step` + skills catalog 注入**模式（有變才重發 `<system-reminder>`）是 dynamic tool/prompt 組裝的好範例。
14. **AGENTS.md 的「foundation over blast radius」**態度本身值得借鏡：在沒有外部 consumer 前，優先正確架構而非相容 shim — 但 Drill 若以 DSH 為基礎，就要自己承擔這條原則的反面（API 隨時會動）。
15. **Python SDK + JSON-RPC stdio 模型**（`DeepSeekHarness` subprocess class）與 Drill 的 Python research backend 整合路徑一致 — 可作為「harness 當 subprocess 用」的官方參考實作。

---

### 附錄：證據索引（本文件引用的關鍵 repo 路徑）

| 主題 | 路徑（commit `47f9438`） |
|---|---|
| 定位/授權/預覽 | `README.md` L5-L11；GitHub API（MIT、2026-08-13、no release） |
| Vendored Cordis | `vendor/README.md` L3-L23 |
| Pre-release stance | `AGENTS.md` L5-L7 |
| 架構/turn flow/seam | `docs/architecture.md` L9-L13、L63-L102 |
| Package groups | `packages/README.md` L11-L59 |
| Web seam 範例 | `packages/web/web/src/index.ts` L1-L66；`web-search-deepseek/src/index.ts` L1-L20；`tool-web/src/index.ts` L2、L24 |
| Session format v0 | `packages/core/session/src/types.ts` L56；`index.ts` L101-L102 |
| Tool pipeline | `docs/tool-execution-pipeline.md`；`core/tools/src/index.ts` L1037 |
| Subagent depth | `subagent/subagent/src/child-agent.ts` L48-L54；`tool-subagent/src/index.ts` L98 |
| Sandbox | `packages/sandbox/sandbox/README.md`；`sandbox-local/src/index.ts` L2、L69、L85-L86 |
| LLM 預設模型 | `packages/llm/llm-deepseek/README.md` L38-L42 |
| Preset 檔案 | `apps/cli/config/agent-presets/{standard,minimal,code,cordis}/agent.cordis.yml` |
| UI 消耗品註記 | `packages/client/AGENTS.md` L48 |
| SDK protocol 限制 | `packages/sdk/protocol/README.md` |
| MCP client | `packages/mcp/mcp-client/README.md` |
| 全文搜尋 opt-in | `packages/bundle/base/cordis.patch.yml` L109-L121 |
| Schedule（session-local） | `packages/schedule/schedule/README.md` |

> 調查方法：deepwiki MCP（14 題分主題查證）＋ gh CLI / 本地 clone（`/tmp/opencode/dsh` @ `47f943859bef60e4160492346772ded9b24f765a`）逐檔驗證。所有引用皆可回溯至 repo。
