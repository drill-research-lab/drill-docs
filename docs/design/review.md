# Design: Review Mode

> 對應 manpage.md「review mode」：審查產生的結果、或審查其他東西。
> Agent: **reviewer**

## 定位

- **review 產生的結果**（論文、報告、研究發現）
- 或 **review 其他東西**（程式碼、實驗、設計）
- **可以被其他 agent 呼叫**，在**無污染上下文**的情況驗證事實
- **anti 幻覺**核心機制

## Agent 設計

### reviewer

- 負責**事實驗證、品質審查、anti-hallucination**
- 參考：pi-subagents 的 reviewer/oracle builtin agents + portos-wang 的 7-agent reviewer 套件
- **核心工具**：
  - `verify_claims(claims, sources)` — 事實查核
  - `cross_check(statement)` — 多源交叉驗證
  - `review_document(doc, rubric)` — 品質審查（0-100 rubric）
  - `check_citations(bib)` — 引用驗證（pi-bib 模式）
  - `suggest_revisions(feedback)` — 修改建議
- **可以被 call**：**任何 agent**（這是它的關鍵設計——clean-context 驗證服務）

## 核心設計：無污染上下文（clean-context）

這是 review mode 最重要的特性：

```
普通 agent 執行：  [完整對話歷史 + 任務 + 工具結果] → LLM → 輸出
reviewer 執行：   [只有「要驗證的 claim + 驗證用 sources」] → 新 LLM call → 結論
```

- **新開一個乾淨的 agent session**（`fully-isolated` context mode）
- 只帶：要驗證的聲明 + 驗證所需的原始來源
- 不帶：原本 agent 的推論、假設、對話歷史
- 效果：**獨立驗證**，不被原 agent 的 bias 污染 → anti-幻覺

## 驗證流程

```
Claim 提出（任何 agent）
    │
    ▼
reviewer 收到 [claim + sources]
    │
    ├─ 有 sources → 逐條比對（claim vs source）
    ├─ 無 sources → 搜尋（Semantic Scholar / arXiv / OpenAlex / web）
    │
    ▼
結論：VERIFIED / CONTRADICTED / UNVERIFIABLE（附證據）
    │
    ▼
回傳給呼叫者（sync block，等結論）
```

### 驗證分級

| 等級 | 意義 |
|---|---|
| **VERIFIED** | claim 與多個獨立來源一致 |
| **PARTIALLY SUPPORTED** | 部分證據支持，部分衝突 |
| **CONTRADICTED** | 有來源明確反駁 |
| **UNVERIFIABLE** | 找不到可驗證來源 |

## 品質審查（review rubric）

- 參考 portos-wang 的 7-agent 多視角 review：
  - 正確性（factual correctness）
  - 完整性（coverage）
  - 一致性（internal consistency）
  - 原創性（novelty）
  - 可重現性（reproducibility，實驗類）
  - 寫作品質（clarity / structure）
  - 引用品質（citation validity）
- 每個視角獨立 agent + 0-100 rubric + 具體建議

## 同步 vs 非同步

| 情境 | 模式 | 理由 |
|---|---|---|
| agent 想繼續，需要驗證結論 | **sync（block）** | anti-幻覺通常要等結論才能繼續 |
| 長驗證（30min+ 查證）| **async** | 不卡死主流程；完成後通知 |
| 論文送審 | **async**（人類審查節點）| 需人類介入 |

→ 需要 **fast review（sync，秒級）** 與 **slow review（async，分鐘級）** 兩種。

## UI 需求

- **審查報告檢視**：驗證結果、證據引用、等級
- **claim-level 標記**：在文件/回答中標出每個 claim 的驗證狀態（綠/黃/紅）
- **對照檢視**：claim ↔ source 並排（side-by-side）
- **審查工作台**：多視角 review 結果彙整（portos-wang 風格）
- **送審流程**：從 paper mode 送初稿 → 追蹤 review 進度

## 整合點

- **Paper**：送審入口、citation 驗證
- **Normal**：orchestrator 隨時 call reviewer 驗證事實
- **Research**：實驗結果 / 程式碼審查
- **Wiki**：知識庫內容的事實查核（librarian 可呼叫）

## 開放問題

1. ✅ 已解決（2026-08-16）：reviewer model **可設定**——預設同主力 model，setting 頁可 override（獨立性主要來自 clean-context；換 model 是加強選項）
2. fast/slow review 的閾值？（多久算 slow？）
3. claim 驗證結果要不要回寫知識庫？（累積「已驗證事實」）
4. 人類審查節點：論文送審需要人 approve 嗎？
