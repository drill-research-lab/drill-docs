# Decisions — 專案決策紀錄

> 記錄 Drill 專案的策略決策與釐清項目（2026-08-15 起）。
> 這些是跨文件的「為什麼」，mode 細節在各 design docs。

---

## D1. 雙軌執行策略（2026-08-15）

**決策**：Drill 以**雙軌並行**開發：
- **Track A：PI-based**（備用 / 個人線）
- **Track B：DSH-based**（團隊主線，zh-TW fork 為基底）

**理由**：
- 兩條軌道最終滿足同一份 spec（drill-docs 是共同契約），雙軌是同一設計的兩次驗證
- PI 生態 30+ plugins（mcp-adapter / web-access / hermes-memory / paper tools）DSH 都還沒有 → PI-based 產出可當 DSH 實作參考
- DSH 是給教授看的基底；PI-based 是保留選項

**Issue 分軌**：以 label 標軌道而非硬拆
- `track-a` = PI-based
- `track-b` = DSH-based
- `both-tracks` = 兩邊都要（實作不同，各開一份）

---

## D2. 自進化作為完成機制（2026-08-15）

**決策**：DSH 的自進化（cordis preset，self-referential meta-agent）是專案的核心**生產方式**，不是加速器。

**流程**：
```
1. 團隊（阿柏等）在 zh-TW fork 上搞定「用戶隔離 + 驗證」← 安全層必須人工做
2. 把 Drill 的 spec（drill-docs）丟進去
3. DSH 自進化：讀 spec → 自己生成/調整 plugin → 完成剩餘功能
```

**邊界**：
- 自進化負責：照 spec 生成 UI 組件 / 工具 / 一般功能
- 人工負責：auth、sandbox 等需要安全考量的核心層
- 呼應 archive/disscuss.md 的「蒸餾」理念：調查 → 抽象 → 沉澱成 template / agent 行為

---

## D3. storage = user workspace + monitor（2026-08-15）

**決策**：storage 不是基礎設施層概念（Postgres/MinIO 等），而是：
- **每個 user 的 agent workspace**：檔案放哪、配額多少
- **磁碟用量監控**：實際用了多少
- **UI 顯示**：system-status 頁面顯示每個 user 的用量

**橫跨的元件**：
- 用戶隔離面：per-user workspace 目錄 / 配額
- system-status 面：用量監控 + UI 顯示
- 兩邊軌道都要（both-tracks），因 DSH（ctx.workspaceRegistry，無 user 維度）與 PI（session-per-cwd，無配額）模型不同

---

## D4. 國科會報告產生器 = paper mode 的第一個 trace bullet（2026-08-15）

**決策**：老紀要的第一個東西（國科會 AI slop 報告產生器）**不是獨立專案**，而是 paper mode 的 **template + skills** 組合，也是專案第一發 trace bullet。

**trace bullet 概念**：建立端到端骨幹（輸入 → LLM → LaTeX → PDF → 網頁）確認彈道能通，再沿著彈道把每個環節做扎實。

**詳細設計**：見 [features/paper/spec.md](features/paper/spec.md) §「第一個垂直切片」。

---

## D5. 部署位置（2026-08-15）

**決策**：部署目標為開放選項，不阻塞現階段工作：
- **spark（DGX Spark）本體只是 LLM provider**（DeepSeek-V4-Flash @ `:8000/v1`）— 不部署平台本體
- **Drill 平台本體**（DSH/PI-based backend + frontend）部署於：
  - 首選：**homelab**
  - 備選：**系上**（islab.local，有人協助）
- 部署位置不影響架構設計（只要 spark `:8000/v1` 能從部署地連到）
- 部署工作由 g36_maid 負責（SPARK 目錄）

---

## D6. 分工（2026-08-15）

**決策**：分工未完全定案，但已澄清：
- **不是全部給阿柏**——阿柏負責部分（auth/sandbox 待確認邊界）
- **部署由 g36_maid 做**（SPARK 目錄）
- Sherlock / NZ 的角色待定
- 分工確認後以 GitHub issues 呈現（待 assign）

---

## D7. 開源替代品直接用；license 不列為決策因素（2026-08-16）

**決策**：
- reference.md 的外部專案是**研究參考**（學設計、理解機制）
- 實作時**若有開源替代品，直接拿來用 / 整合 / 重寫皆可**——不需要因為 license 繞路
- Drill 本身要開源，因此整合 OSS 不構成障礙

**理由**：license 約束的是「重複分發程式碼」；Drill 開源釋出，整合任何 OSS 都有可行路徑。文件中的 license 註記（reference.md）僅為參考專案的事實描述，不是 Drill 的決策輸入。

**注意**：設計文件中出現的「fork/embed 某專案」字樣（如 paper 的「Fork Overleaf CE」選項，現於 features/paper/）是調查過程的選項枚舉，**不是產品需求**；原始需求以 index.md（原 manpage.md）為準（例：paper mode =「自帶 latex builder / preview / pdf viewer，像 overleaf 一樣」，非「fork overleaf」）。

---

## 待釐清項目

| # | 項目 | 狀態 |
|---|---|---|
| 1 | 分工邊界（阿柏 / Sherlock / NZ / g36_maid 各做什麼）| ⏸ 待定 |
| 2 | PI-based 備用線：並行 or 等 DSH 瓶頸才啟動 | ⏸ 待定 |
| 3 | 自進化的 spec 輸入格式（markdown？結構化？）| ⏸ 待定 |
| 4 | 部署位置最終決定（homelab vs 系上）| ⏸ 後再決定 |
