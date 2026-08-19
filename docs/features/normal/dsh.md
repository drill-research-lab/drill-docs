# Chat — Track B（DSH-based）註記

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/dsh.md](../../tracks/dsh.md)。
> 深度調查：[reference/dsh.md](../../reference/dsh.md)。

## Subagent 派遣：實作選項（自 spec 移入）

| 選項 | 說明 | 取捨 |
|---|---|---|
| **DSH subagents seam** | `ctx.subagents` capability seam，providers：`subagent-spawn-in-process` / `subagent-fork-in-process` / `subagent-acp` / `subagent-codex` / `subagent-claude-code` / `subagent-dsh-sdk`；`delegationDepth` monotone + `maxSubagentDepth` | 最完整（多 provider 可替換）；DSH fork 內建，零額外成本 |

## Agent presets 分層參考（自 spec 移入；DeepWiki 實證 2026-08-16）

DSH 預設四個 agent preset（定義在 `agent.cordis.yml`；原始調查 [reference/dsh.md](../../reference/dsh.md) §9）——對 Drill 的 agent 分層是直接可抄的參考：

| DSH preset | 內容 | Drill 對應 |
|---|---|---|
| `standard` | 完整 agent：persona + instructions + 全 toolset（e2e 實測 23 個 tools，含 `ralph`、`workflow`、`subagent`/`subagent_fork`、`send_message`、`interrupt_agent`…） | orchestrator / researcher / writer 等主力 agents |
| `minimal` | 固定 prompt 雙 tool agent（僅 persistent bash + str_replace_editor），`complete: true` + 無 runtime context、無 compaction | pipeline 的 **simple llm / mini agent 節點**（輕量、可預測） |
| `code` | standard 全部 + `tool-presentation` row → Code Mode（model 寫 TS 打 generated SDK，五個 round trip 變一個） | researcher 的程式密集任務（可選強化） |
| `cordis` | standard 全部 + self-referential toolset（`cordis_mount` 直接對 live runtime 跑 model 寫的 JS；官方定位「建 custom agent presets + runtime inspection + plugin 實驗」）；TRUST「等同 shell access」 | **D2 自進化 meta-agent**（讀 spec → 生成/調整 plugin → 沉澱成 preset） |

**Preset 機制**（值得直接沿用）：preset = 目錄 + `agent.cordis.yml`；roster 每 process mount 一次（standing scope），session 以 scope parentage 加入；service row 必須在 `isolate` realm 內否則 mount reject；authoring 只允許 copy（`ctx.agentPresets.copy`）；id 限制 `[a-z0-9][a-z0-9-]*`。

**Drill 的 agent 分層**：主力 agents（orchestrator/researcher/writer/reviewer/librarian）以 `standard` 為基底做 custom presets；pipeline 節點 agent 以 `minimal` 為基底；D2 自進化用 `cordis` preset（安全層——用戶隔離/驗證——先人工做掉，見 [tracks/dsh.md](../../tracks/dsh.md)）。

## seam / plugin 對應

| spec 需求 | DSH 對應 |
|---|---|
| `spawn_subagent` sync/async | `ctx.subagents` seam（spawn/fork providers） |
| context 三模式 | spawn≈fresh、fork=繼承歷史；`shared-project` 需以 context injection 實作 |
| 權限繼承 | Cordis isolate realm + preset trust 層級 |
| orchestrator persona | custom preset（copy `standard` 起改） |
