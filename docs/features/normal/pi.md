# Chat — Track A（PI-based）設計

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/pi.md](../../tracks/pi.md)。
> 原則：最大化複用 PI 生態系（[reference/pi-ecosystem.md](../../reference/pi-ecosystem.md)），gap 才自建。

## Subagent 派遣：實作選項（自 spec 移入）

| 選項 | 說明 | 取捨 |
|---|---|---|
| **pi-subagents**（現成，主力候選） | `subagent()` tool + `workflowScript`（`runs.run` / `runs.all` 平行 / branching / retry / gate / aggregation）+ agent = MD + YAML frontmatter；context 模式 **fresh / fork**；`contact_supervisor`（child→parent mid-execution）；`maxSubagentDepth` 預設 2；worktree isolation | 最快；builtin agents（scout/researcher/worker/reviewer/oracle/delegate）是 coding 導向，需替換成 Drill agents（librarian/researcher/writer/reviewer/pipeline builder）；context 模式只有 2 種，spec 要 3 種 |
| **oh-my-openagent task tool**（參考） | category-based routing + `omo.jsonc` agent 定義（`OmoAgentDefSchema`：model/tools/max_depth/allowed_subagents…）；permission 繼承模型 | 架構最合（agent 定義 schema 可直接抄）；但 omo 是獨立 harness 不是 library——抄設計不引套件 |
| **自建** | 在 pi-agent-core 上做 `spawn_subagent` tool（`Agent` class + `agentLoop` + extension API） | 完全掌控三種 context 隔離模式；工作量大 |

### Context mode 對照（spec 三種 vs pi-subagents 兩種）

| spec（Drill） | pi-subagents | 缺口 |
|---|---|---|
| `fully-isolated` | `fresh`（最小 context 起） | ≈ 已覆蓋 |
| `shared-project` | —（無此概念） | **gap**：需擴充（把 project 知識/AGENTS.md 注入 child） |
| `fresh-inherits-task` | `fresh` + task 傳遞 | ≈ 已覆蓋（`fork` 是另一種：繼承 parent 歷史，spec 未列） |

### 建議路線（草稿，待驗證）

以 **pi-subagents 為基底** + 自訂 Drill agents（MD+frontmatter 格式現成）+ 擴充 `shared-project` context mode（作為 pi extension 實作）。

## 記憶 / 長期偏好（未定，見 spec 整合點）

- 候選：pi-hermes-memory（L1-L4、FTS5、無 vector）或 pi-knowledge（RAG）——選型見 [wiki/pi.md](../wiki/pi.md) 的索引層分析（共用同一決策）
- 深層方向：HERMES **skill 機制**（程序性知識自演化）→ [skills/pi.md](../skills/pi.md)

## 自建 gap 清單

1. `shared-project` context mode（pi-subagents 擴充）
2. Drill agent 定義集（librarian / researcher / writer / reviewer / pipeline builder / mini agent / simple llm——MD + YAML frontmatter）
3. subagent 活動指示器 / agent 呼叫圖 UI（前端，全功能共用）
