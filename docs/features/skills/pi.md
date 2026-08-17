# Skills 管理頁 — Track A（PI-based）設計

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/pi.md](../../tracks/pi.md)。
> 原則：最大化複用 PI 生態系（[reference/pi-ecosystem.md](../../reference/pi-ecosystem.md)），gap 才自建。

## 複用組合

| 需求 | 複用 |
|---|---|
| SKILL.md 格式 + progressive disclosure | PI 原生（skills 是 pi-package `pi` field 的一等公民）|
| Import（npm / Git） | `pi install npm:@foo/skill` / `pi install git:github.com/...`（~/.pi/agent/npm/）|
| `skill` tool（create/view/patch/edit/delete） | **pi-hermes-memory** 的 skill tool（HERMES 移植）|
| 4-scope 發現 | 抄 oh-my-openagent 模型（project > opencode > user > global）|
| CLAUDE.md → SKILL.md 轉換 | portos-wang conversion skill（未經 DeepWiki 驗證）|

## 自建 gap 清單

1. Web 管理頁（列表/add/import/edit/versioning/preview——PI 是檔案系統形態，UI 全自建）
2. 自動演化 + 審批閘門 + Curator（spec 參考模型；pi-hermes-memory 只有 skill tool，演化機制要自建——可移植 HERMES 設計）
3. skill-embedded MCP（frontmatter 宣告 `mcp:`——omo 的三層之一，PI 無此機制）
