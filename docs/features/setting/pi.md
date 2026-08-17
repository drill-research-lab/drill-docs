# Setting 頁 — Track A（PI-based）設計

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/pi.md](../../tracks/pi.md)。
> 原則：最大化複用 PI 生態系（[reference/pi-ecosystem.md](../../reference/pi-ecosystem.md)），gap 才自建。

## 複用組合

| 設定面 | 複用 |
|---|---|
| LLM providers / API keys / fallback chain | **pi-ai**（47 provider factory + credential resolution）——設定頁前端的後端現成 |
| 參數（temperature / max_tokens / reasoning） | pi-ai 的 per-call 參數面 |
| permission / 審核 | PI in-tree `permission-gate` extension 模式 |
| 記憶/KB 設定 | 對應 plugin 的 config（pi-knowledge / pi-hermes-memory——見 [wiki/pi.md](../wiki/pi.md)）|

## 自建 gap 清單

1. 設定 UI + 持久化 config API（PI 無 web 設定面）
2. 多使用者/角色/LDAP（用戶隔離是必須功能，時程未定）
3. 專案管理（projects 列表/預設）
4. 通知（Discord webhook / email）
