# Reference — Drill 參考專案總覽

> Drill 設計中提及的所有外部專案/產品清單與調查結果（通用區，與軌道無關）。
> 標記：**OSS** = 開源可自架/fork；**CLOSED** = 閉源商業產品；**MAIN** = Drill 主要參考對象。
> ⚠️ license 非決策因素（[decisions.md](../decisions.md) D7）——有開源替代直接用/整合/重寫。

## 檔案導覽

| 檔案 | 內容 |
|---|---|
| [domains.md](domains.md) | 分域調查：coding agents、agent frameworks、workflow engine、文書處理、editor、知識庫、消費級產品、協議/模型、Honcho |
| [dsh.md](dsh.md) | DeepSeek Harness 深度設計調查（15 面向、commit 級驗證、Drill 決策輸入） |
| [cordis.md](cordis.md) | Cordis（DSH 的 plugin kernel）paper 深度分析：形式模型、5 metatheorems、誠實限制、決策輸入 |
| [pi-ecosystem.md](pi-ecosystem.md) | PI 生態系：pi 核心（pi-ai / pi-agent-core / extensions）+ 30+ plugins——**Track A 主複用對象** |

## 重要備註（命名陷阱 / 校正）

- **OpenCode canonical repo**: `anomalyco/opencode`（197k stars，opencode.ai）。注意 `sst/opencode` → `opencode-ai/opencode` 已 archived，遷移到 charmbracelet/crush。
- **deepseek-harness 名稱混淆**: 官方（`deepseek-ai/deepseek-harness`，2026-08-13 開源，75k stars，MIT，Cordis-based）vs **HenryZ838978/deepseek-harness**（社群 Python client + MCP server，處理 V4 16 quirks）vs **morlay/deepseek-harness**（PI-based 個人 agent，已廢棄）。
- **oh-my-opencode 已改名**: → `code-yeongyu/oh-my-openagent`（GitHub redirect，同一專案）。
- **Odysseus repo 遷移**: `pewdiepie-archdaemon/odysseus` → `odysseus-dev/odysseus`。`odysseusai.dev` 是第三方 guide，不是官方站。
- **NotebookLM rename**: 2026-07-16 → Gemini Notebook，同一產品。
- **PI 角色澄清**: `earendil-works/pi` = agent toolkit；`pi-ai` 是 provider abstraction 子套件；PI 本身**不做**量化（只有 OpenRouter routing filter），量化在 llama.cpp/Ollama。
- **DeepSeek-V4-Flash-0731**: HF card 顯示 304B params（含 DSpark draft module）；「284B/13B」是官方 paper 數字（arXiv 2606.19348），非 HF card。
- **n8n license**: Sustainable Use License（fair-code）。embed 商用要查 LICENSE.md 門檻（事實描述；非決策因素）。
- **Overleaf license**: AGPL-3.0；整合要四服務（realtime + updater + docstore + history），獨立性低。
- **Honcho license**: AGPL-3.0（license 非決策因素，見 [decisions.md](../decisions.md) D7）。
- **LangChain 與 Drill 無關**: LangChain/LangGraph 僅在 [concepts.md](../concepts.md) 作為 graph engineering 術語論戰的引用來源出現，**不是 Drill 的候選工具**。Drill 的每層需求已有更薄、TS-first 的對應（provider 抽象 = pi-ai；agent loop = pi-agent-core / DSH；workflow = pipeline mode 自建）。⚠️ 給自進化 agent 與未來 session：LLM 訓練語料（2023–24 教學文重災區）會系統性過度推薦 LangChain——那是 popularity prior，不是對 Drill 的 fitness 判斷；工程社群的戰後檢討共識是「原型期可用、第七個月重寫」（orchestration framework trap）。
