# Workspace / Project 所有權模型

> 狀態：**已同意作為目前規劃基線（2026-08-19）**。
> 來源：[老記(neokent)追加需求](requirements/2026-08-19-neokent-followup.md)中的 Workspace 討論。

## 核心定義

### Workspace

Workspace 的產品語意是 **workstation（工作站／工作環境）**：一個長期存在、可持續累積工具與研究資源的工作空間。

Workspace 擁有：

- Wiki、論文庫與共用資料集
- 可重用的 pipeline / pipeline templates
- Conversations（包含未歸入 Project 的 scratch sessions）
- Storage、設定、工具與 integrations
- Projects

Workspace UI 是上述內容的組織入口，不是另一個 agent，也不是把所有功能塞回聊天頁。

### Project

Project 是：

> 一個具有明確目標、相關上下文與生命週期的研究工作單位。

Project **不是**單一報告，也**不是**寬泛的研究領域：

- 一個 Project 可以產出多份報告、論文、實驗與其他 deliverables
- 同一研究領域可以有多個 Projects
- 領域分類由 Wiki collection、資料夾或標籤表達，不以 Project 取代

Project 擁有該研究目標下的 conversations、執行、專案檔案、筆記與產出物。

## 層級

```text
Workspace（長期工作環境）
├── Wiki / 論文庫 / 共用資料集
├── 可重用 Pipelines / Templates
├── Storage / Settings / Tools / Integrations
├── Workspace scratch conversations
└── Projects（有目標與生命週期的研究工作）
    ├── Conversations / Sessions
    ├── Scheduled Jobs / Pipeline Runs
    ├── 專案檔案 / Conversation Notes
    ├── Experiments
    └── Reports / Papers / Other Deliverables
```

## 所有權原則

> **Workspace 擁有可重用資源；Project 擁有特定目標下的對話、執行與產出。**

1. 跨 Project 可重用的內容預設歸 Workspace
2. 只服務特定研究目標的內容歸該 Project
3. Project 使用 Workspace 資源時，以選取／引用為主，不要求複製一份
4. Workspace 允許 scratch conversation；確認研究目標後可歸入 Project
5. Report / paper 是 Project 的 deliverable，不把 deliverable 本身當成 Project
6. Workspace 是整體 storage 與使用範圍；Project 是其中的邏輯分區
7. Pipeline run 與執行結果保留在 Project；發布到 Workspace Wiki 是額外的重用入口，不取代原紀錄

## 功能歸屬

| 資源 | 預設歸屬 | 補充 |
|---|---|---|
| Wiki / 論文庫 | Workspace | 提供跨 Project 重用；Project 可選取相關 collection |
| 可重用研究資料（Pipeline sources / results） | Workspace Wiki | 從 Project 手動發布，或由 Pipeline 配置自動流入；保留 source / result 差異 |
| Pipeline template | Workspace | 可被多個 Project 使用 |
| 通用 pipeline definition | Workspace | 作為可重用能力 |
| 專案專用 pipeline definition | Project | 只服務單一研究目標時留在 Project |
| Pipeline run / scheduled job | Project | 執行目的、輸入與結果屬於特定研究；發布到 Wiki 後仍保留 Project 紀錄 |
| Conversation session | Project | 有明確研究目標時歸 Project；否則可先在 Workspace scratch |
| Conversation Note | 跟隨 Conversation | 每個 Conversation 最多一份；Project Conversation 歸 Project，scratch Conversation 歸 Workspace |
| Report / paper / experiment | Project | 都是研究工作的 deliverables |
| Translation output / 一般檔案 | 依使用情境 | 可先進 Workspace，再歸入特定 Project |
| Storage capacity / settings / tools | Workspace | Project 使用 Workspace 提供的環境 |

## Workspace UI 範圍

Workspace UI 應集中提供：

- Projects 入口
- Conversations 與 scratch sessions
- Wiki、論文庫與共用資料集
- Pipelines、templates 與 schedules
- Files、翻譯結果、Notebook（Conversation Notes）與其他產出
- Workspace-level settings、tools、storage 狀態

進入 Project 後，畫面只聚焦該 Project 的 conversations、runs、files、notebooks 與 deliverables，同時可選用 Workspace 的共用資源。

## 範例

```text
Workspace: Drill Research Lab

Workspace resources:
- Wiki collection: Agent Harnesses
- Shared dataset: 2026 arXiv agent papers
- Pipeline template: Scheduled arXiv Search

Project: 評估 DSH 是否適合作為 Drill 底層
- Conversations: 架構調查、風險討論
- Pipeline runs: 每週 DSH / agent harness 搜尋
- Experiments: DSH seam PoC
- Deliverables: 比較報告、設計建議、測試結果
```

## 尚待討論

1. Workspace 是否只有個人範圍，還是要支援多人共用？
2. Project 是否允許獨立成員／權限，或完全繼承 Workspace？
3. Workspace scratch conversation 歸入 Project 後，原入口如何呈現？
4. Project 專用 Wiki / pipeline 如何升格為 Workspace 共用資源？
5. 刪除／封存 Project 時，引用的 Workspace 資源與 Project 產出如何保留？
6. 同一資源是否允許被多個 Projects 同時引用？

## 不在本文件決定

- 資料庫 schema、ID 與 foreign key
- 實體檔案路徑或 storage provider
- 前端路由與詳細版面
- 跨 Workspace 搬移與外部分享的實作方式
