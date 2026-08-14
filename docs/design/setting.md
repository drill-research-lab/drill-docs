# Design: Setting 頁

> 對應 manpage.md「setting page」：just any setting。

## 定位

- 所有設定集中管理
- 「just any setting」——涵蓋 drill 各層

## 設定分類

### 1. 帳號 / 使用者
- 個人資料（名稱、email、偏好語言）
- 密碼 / 2FA
- 多使用者（若啟用）：邀請、角色、權限
- **LDAP 整合**（若實驗室情境）

### 2. LLM / Providers
- 模型設定：
  - 主力模型（DeepSeek-V4-Flash-0731 @ spark `:8000/v1`）
  - fallback chain（有序）
  - 各 mode 的 model override（reviewer 用獨立 model？）
- Provider API keys（OpenAI / Anthropic / Google / Perplexity / Exa…）
- **路由策略**：
  - 機敏資料 → 僅本地模型（國科會合規）
  - 非機敏 → 可外部
- 參數：temperature / top_p / max_tokens / reasoning level

### 3. 專案（Projects）
- 專案列表 / 建立 / 刪除
- 每個專案的：知識庫、workspace、agents、pipelines
- 專案預設（model、sandbox、memory 設定）

### 4. 記憶 / 知識庫
- memory provider 選擇（pi-hermes-memory / pi-knowledge / Honcho…）
- 記憶 scope（global / project / user）
- 知識庫管理（重建索引、清空）

### 5. 沙盒 / 執行環境
- sandbox backend（Docker / microVM / 雲端）
- 資源配額（CPU / memory / walltime）
- 網路白名單
- 未受信任 MCP server 是否在沙盒跑

### 6. 審核 / 安全
- **自主程度**：審核節點開關（全自主 vs 每步確認）
- 敏感動作白名單 / 黑名單
- subagent 深度限制（max_depth）
- 權限繼承規則

### 7. 外觀 / 偏好
- 主題（dark / light / auto）
- 語言（zh-TW / en）
- 編輯器設定（tab size、font）

### 8. 通知
- Discord webhook 整合（disscuss.md 有提 Discord）
- 電子郵件通知（實驗完成、review 完成）
- 事件通知設定

## UI 需求

- 分頁式設定（左側分類導航）
- 即時儲存 / 套用
- 進階設定顯示（隱藏危險選項）
- 匯出 / 匯入設定（備份）

## 開放問題

1. 設定要 per-user 還是 per-project 分層？（大部分兩者都有：global default + project override）
2. LDAP 是 v1 還是 v2？
3. Discord 通知是 v1 嗎？
