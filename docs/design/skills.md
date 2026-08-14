# Design: Skills 管理頁

> 對應 manpage.md「skills」：add / import / edit skills files。

## 定位

- 管理 Drill 的 **skills 檔案**（SKILL.md 格式）
- 操作：**add / import / edit**

## Skills 是什麼

- **SKILL.md** = Agent Skills 標準格式（YAML frontmatter + markdown body）
- Skill 是 agent 的**能力包**：progressive disclosure（需要時才載入，省 context）
- 參考生態：
  - PI：SKILL.md-based，`pi install npm:` 安裝（`docs/reference.md` §2, §9）
  - oh-my-openagent：4-scope 發現（project > opencode > user > global）
  - HERMES：skill 自動演化（`plugins/memory/skills.py`）
  - pi-hermes-memory：`skill` tool（create/view/patch/edit/delete）

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
- **自動演化**（v2）：HERMES-style 背景 review → 自動建立/更新 skills

## UI 需求

- skill 列表 + 詳細檢視
- frontmatter 編輯（結構化 form，非純文字）
- import 流程（來源選擇 → 預覽 → 確認）
- 衝突處理（同名 skill 覆蓋/保留）

## 開放問題

1. skill 的自動演化（agent 觀察到重複模式 → 自動建 skill）v1 還是 v2？
2. 需要 skill versioning / 依賴管理嗎？（SKILL.md 目前無內建版本語義）
3. skill 權限：哪些 skill 可以被 agent 自動建立？哪些要人 approve？
