# Conversation Note / Notebook

> 狀態：**已同意作為目前產品規格（2026-08-19）**。
> 來源：[老記(neokent)追加需求](requirements/2026-08-19-neokent-followup.md)中的「筆記本」討論。

## 定位

每個 Conversation 自帶一份小型 Markdown 文件，名稱為 **Conversation Note**。它整理目前討論，供使用者回顧，也供其他 agent 快速理解這段 Conversation。

Conversation Note 不依賴「session 結束」：Conversation 沒有可靠的自然結束事件，Note 可在任何時間由使用者產生或更新。

## 核心規格

- 每個 Conversation 最多一份 Conversation Note
- 尚未建立時提供「產生 Note」
- 已建立後提供「更新 Note」
- Note 總結目前 Conversation，而不是自由筆記的空白頁
- 使用者可以請 LLM 修正／更新 Note
- 必要時，使用者可以直接修改 Note
- 使用者可以選擇重新產生 Note
- Note 保存為 Markdown，供使用者與其他 agent 閱讀
- Note 連回原始 Conversation，方便查閱完整上下文

## 建議內容

```markdown
# Conversation Note

## 討論摘要

## 已確認決策

## 重要資料與產出

## 尚未決定

## 下一步
```

此結構是預設整理方向，不要求所有 Conversation 都必須產生每個區塊。

## Notebook

Notebook 是 **Conversation Notes 的集中瀏覽入口**：

```text
Notebook
├── Project A
│   ├── Conversation 1 Note
│   └── Conversation 2 Note
└── Workspace Scratch
    └── Conversation 3 Note
```

Notebook：

- 不是第二套 Wiki
- 不是 NotebookLM 式論文／來源資料庫
- 不是以自由寫作為核心的筆記工具
- 不複製完整 Conversation，只集中整理與導覽 Conversation Notes

## 與 Wiki 的區別

| 區域 | 內容 | 主要用途 |
|---|---|---|
| Wiki / 論文庫 | 外部來源、原始 PDF、轉換後 Markdown、Pipeline sources / results | 查資料、引用證據、供 librarian 搜尋 |
| Notebook | Conversation Notes | 回顧討論、決策、未決事項與下一步 |

## 所有權

- Project Conversation 的 Note 歸該 Project
- Workspace scratch Conversation 的 Note 歸 Workspace
- Notebook 依 Workspace / Project 範圍整理 Notes，不改變 Note 原本的歸屬

## 與 agent 的關係

- Conversation Note 可提供給其他 agent 作為這段討論的精簡背景
- Note 是輔助資料，不取代原始 Conversation
- 是否自動提供給 agent、由哪個 agent 更新、如何處理局部修改，留待後續工具與 context 設計

## 暫不決定

- LLM 產生、修正與重新產生 Note 的 tool schema
- 更新 Note 時採增量修改或完整重寫
- Note 的版本歷史與衝突處理
- 是否在切換／封存 Conversation 時提示更新 Note
- 哪些 agent 預設可讀取或修改 Note
