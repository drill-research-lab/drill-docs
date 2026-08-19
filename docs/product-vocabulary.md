# Product Vocabulary（產品功能與名稱）

> 狀態：**命名基線，仍可調整（2026-08-19）**。
> 本文件只定義產品層級與使用者看到的名稱；底層 preset、tool 與權限配置由各軌道文件定義。

## 命名層級

| 層級 | 用途 | 例子 |
|---|---|---|
| Product container | 長期工作環境與研究工作 | Workspace / Project |
| Page / Surface | 使用者進入的主要功能 | Chat / Library / Lab |
| Product object | 頁面中管理的內容 | Conversation / Document / Pipeline Run |
| Agent | 執行工作的角色 | Orchestrator / Librarian / Writer |
| Agent / Mode | Chat 中選擇的工作方式 | Standard / Orchestrator / Minimal |

頁面、物件、agent 與 mode 不使用同一個名稱層級互相替代。

## 主要頁面名稱（暫定）

| 產品名稱 | Legacy path | 定義 | 狀態 |
|---|---|---|---|
| **Chat** | `normal` | Conversations 與 agent 互動入口 | 採用 |
| **Library** | `wiki` | Wiki、論文庫、Sources 與可重用研究資料 | 採用；用來與 Notebook 區分 |
| **Pipelines** | `pipeline` | Pipelines、templates、schedules 與 runs | 採用 |
| **Lab** | `research` | 實驗、PoC、程式碼與執行環境 | 採用 |
| **Writing** | `paper` | 與 Writer 協作建立 paper、report、proposal | 暫定；名稱仍可調整 |
| **Review** | `review` | 事實查核與品質審查 | 採用 |
| **Notebook** | 新增 | Conversation Notes 的集中瀏覽入口 | 採用 |
| **Files** | storage UI | Workspace / Project 檔案與產出物 | 採用 |

補充：

- Translation 暫時是候選 built-in 或 Files action，不是正式主要頁面
- Scheduled Search 是 Pipeline template，不是主要頁面
- PDF Viewer 是 Chat / Library / Writing 共用能力，不是主要頁面
- Skills、MCP Servers、Settings、System Status 屬於設定／管理區，不與研究頁面並列

## Chat：Agent / Mode selector

Chat 提供三個**平級**的 built-in 選項：

| 名稱 | 產品定位 | 目前能力輪廓 |
|---|---|---|
| **Standard** | 預設完整工作模式 | 檔案編輯、Shell、檔案與 Web 檢索、Skills、計畫、目標、subagents 與 workflows |
| **Orchestrator** | 較強的協調與自主執行角色 | 以 Standard 為基礎，具有更高權限與更積極的多 agent 協調；精確邊界待定 |
| **Minimal** | 刻意縮小工具面的工作模式 | 只提供完成檔案／程式工作所需的最小工具集合；PI / DSH 的確切工具不同，見軌道設計 |

### Minimal 的產品定位

- Minimal **不是 Advanced，也不是能力較差的降級模式**
- 它與 Standard / Orchestrator 平級，讓使用者主動選擇較小、較可預測的 agent surface
- 較少工具可能減少 prompt/tool schema 負擔、錯誤工具選擇與不必要步驟
- 這不代表 Minimal 對所有模型與任務都更好；效果取決於 model、task、prompt、tool schema 與 harness 的組合
- DSH 官方將 Minimal 定位為只有 persistent bash + file editor 的 benchmark-oriented minimal environment，未聲稱普遍勝過 Standard

參考：

- [DeepSeek Harness developer preview](https://deepseek.com/harness/en/)：Standard / Code / Minimal / Creator 四個 UI modes
- [GitHub Copilot: smarter with fewer tools](https://github.blog/ai-and-ml/github-copilot/how-were-making-github-copilot-smarter-with-fewer-tools/)：縮減預設工具面後，benchmark resolution rate 提升 2–5 percentage points
- [Pi coding agent：Minimal toolset](https://mariozechner.at/posts/2025-11-30-pi-coding-agent/)：以 read / write / edit / bash 四工具作為有效 coding agent 基線
- [How Many Tools Should an LLM Agent See?](https://arxiv.org/html/2605.24660v2)：較短且依 query 調整的工具清單可改善工具選擇

## Temporary Chat

Temporary Chat 是 Conversation 的保存／記憶選項，不是第四個 Agent / Mode：

- 不保存到一般 Conversation history
- 不產生 Conversation Note
- 不進入長期 memory
- 不出現在 Notebook
- 若 Standard / Orchestrator / Minimal 已修改檔案或啟動外部工作，Temporary 不自動撤銷這些效果

## Agent 名稱

| Agent | 主要服務頁面 |
|---|---|
| Orchestrator | Chat（Orchestrator mode） |
| Librarian | Library |
| Pipeline Builder | Pipelines |
| Researcher | Lab |
| Writer | Writing |
| Reviewer | Review |

## 待定名稱／邊界

1. Writing 是否保留，或改成 Writer / Documents / Studio
2. Standard 與 Orchestrator 的精確權限與 delegation 差異
3. Minimal 是否依 model 顯示建議標記，但不改變其平級地位
4. Selector 在 UI 上稱 Agent、Mode，或 Agent / Mode
5. 是否允許 Workspace 為不同 model 設定預設 mode
