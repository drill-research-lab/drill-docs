# Design: Skills 管理頁

> 對應 index.md（原 manpage）「skills」：add / import / edit skills files。

## 定位

- 管理 Drill 的 **skills 檔案**（SKILL.md 格式）
- 操作：**add / import / edit**

## Skills 是什麼

- **SKILL.md** = Agent Skills 標準格式（YAML frontmatter + markdown body）
- Skill 是 agent 的**能力包**：progressive disclosure（需要時才載入，省 context）
- 參考生態：
  - PI：SKILL.md-based，`pi install npm:` 安裝（`docs/reference.md` §2, §9）
  - oh-my-openagent：4-scope 發現（project > opencode > user > global）
  - HERMES：skill 自動演化 + Curator 背景維護（見下方「參考模型」，完整細節 `docs/reference.md` §2）
  - pi-hermes-memory：`skill` tool（create/view/patch/edit/delete）

## 參考模型：HERMES skill 機制（團隊欣賞的方向）

> DeepWiki 實證 2026-08-16。這是 Drill skills 設計最主要的參考。

| 機制 | HERMES 做法 | 對 Drill 的啟示 |
|---|---|---|
| **Progressive disclosure** | 三層：`skills_list()` 輕量 index（~3k tokens）→ `skill_view(name)` 全文 → `skill_view(name, path)` 支援檔按需 | context 省法的具體實作層級 |
| **Skill 結構** | `SKILL.md` + `references/` / `templates/` / `scripts/` / `assets/` | 結構直接沿用 |
| **演化（前景）** | agent 用 `skill_manage`（create/edit/patch/delete/write_file/remove_file）自我改善；觸發：user corrections / 非平凡技巧 / skill 過時 | agent 可自建/自改 skill |
| **演化（背景）** | `_SKILL_REVIEW_PROMPT` 驅動背景 review agent | 背景自我改善的 prompt 模式 |
| **寫入審批** | `skills.write_approval` 閘門：staged pending + `/skills diff` + approve/reject | 對應開放問題「哪些 skill 可自動建立、哪些要人 approve」的現成解法 |
| **保護圈** | bundled / hub-installed / `external_dirs` / pinned / user-owned 不可自動改 | Drill 的 built-in / imported skills 應同樣保護 |
| **Curator（背景維護）** | 閒置觸發（≥7 天 + agent 閒置 ≥2h）；30 天標 stale、90 天 archive（可恢復）、LLM consolidation 合併重複（預設關）；pass 前自動快照可 rollback；三重 guard（pinned 不可刪、背景寫入從嚴、consolidation 刪除限「已被 umbrella 吸收」） | skill 庫的垃圾回收機制——只歸檔不刪除、快照可回滾、背景從嚴 |

## 頁面功能

### 1. 瀏覽 / 列表
- 所有已安裝 skills（名稱、描述、來源、啟用狀態）
- 分類 / 搜尋 / 過濾
- 來源標記：built-in / project / user / global / npm package

### 2. Add（新增）
- 手動建立新 skill（編輯器：frontmatter + body）
- 或用 agent 建立（`/skill:name` 或 natural language → agent 生成 SKILL.md）

### 3. Import（匯入）
- 從 **npm**：`pi install npm:@foo/skill`（pi-package 生態）
- 從 **Git**：`pi install git:github.com/...`
- 從**檔案**：上傳 SKILL.md / 資料夾
- 從 **Claude Code / Codex 生態**：portos-wang 的 conversion skill（CLAUDE.md → SKILL.md 對照）

### 4. Edit（編輯）
- 編輯 frontmatter（name / description / 觸發條件）
- 編輯 body（instructions）
- **versioning**：編輯歷史 / 回滾
- **preview**：skill 生效後的效果

## Skill 結構

```markdown
---
name: my-skill
description: 什麼時候用這個 skill
# 可選 frontmatter：
# models: 指定 model
# mcp: 需要的 MCP servers（skill-embedded MCP，oh-my-openagent 三層之一）
# tools: 允許的工具
---
# My Skill
（instructions body，agent 載入後執行）
```

## 整合點

- **4-scope 發現**（參考 oh-my-openagent）：project > opencode > user > global
- **Agent 使用**：`/skill:name` 或自動觸發（description 匹配）
- **Skills 與 subagent**：child agent 可帶 skills（`load_skills=[...]`）
- **Skills 與 pipeline**：mini agent 節點可設定 skills
- **自動演化**（時程未定）：HERMES-style 演化 + 審批閘門 + Curator 背景維護（見上方參考模型）

## UI 需求

- skill 列表 + 詳細檢視
- frontmatter 編輯（結構化 form，非純文字）
- import 流程（來源選擇 → 預覽 → 確認）
- 衝突處理（同名 skill 覆蓋/保留）

## 開放問題

1. skill 的自動演化（agent 觀察到重複模式 → 自動建 skill）何時做？做的時候要不要直接抄 HERMES 的審批閘門（write_approval + staged pending + diff approve）？
2. 需要 skill versioning / 依賴管理嗎？（SKILL.md 目前無內建版本語義）
3. skill 權限（傾向，未定案）：user 設定 / pinned 的 skill 保護（不可自動改），其餘讓 agent 自由增長——即 HERMES 的保護圈模型；審批閘門為 user 可選開關
