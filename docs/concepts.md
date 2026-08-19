# Concepts — Agent 工程術語階梯

> 解釋 2023–2026 年業界逐步命名的五個 agent 工程層：**Prompt → Context → Harness → Loop → Graph engineering**。
> 這不是五個互相取代的潮流，而是**同一個系統的五個樓層**——每層包裹前一層；下層弱，上層只會把錯誤放大得更華麗。
> 本文件是團隊共同語彙：Drill 的 Pipelines 本質是 graph engineering，node 內是 loop engineering，承載一切的是 harness。
> 基準：2026-08。術語仍在演化（LangChain 官方也自嘲這些詞「存在且其來有自，但終究是 buzzword」）。

---

## 1. 一張表看懂：五層階梯

| 年代 | 層 | 核心問題 | 你在設計什麼 | 命名事件 |
|---|---|---|---|---|
| 2023–24 | **Prompt** | 這句話怎麼問？ | 單一請求的措辭 / 格式 / few-shot | 無單一命名者，早於 agent 時代 |
| 2025 | **Context** | model 這次 call 看到什麼？ | 填進 context window 的所有 token（system prompt、記憶、RAG、tool 定義、tool 輸出） | Tobi Lütke 提議、Karpathy 背書（2025-06） |
| 2026 初 | **Harness** | 這一次執行怎麼跑？ | model 以外的一切：tools / sandbox / retry / 權限 / 子 agent / 觀測 | Vivek Trivedy（2026-03，歸屬有爭議）；Osmani O'Reilly 文（2026-05） |
| 2026/06 | **Loop** | 下一步做什麼、何時停？ | 重複觸發 agent 的系統：trigger / goal / 驗證 / 停止條件 / memory | Boris Cherny 一句話 → Addy Osmani 命名（2026-06-07） |
| 2026/07 | **Graph** | 多個 loop 怎麼接？ | nodes / edges / shared state——多 agent 的拓撲與協調 | Peter Steinberger 推文引爆（2026-07 中），數日內定名 |

**累積法則（nesting rule）**：Harness 實作 loops → 每個 loop step 組裝 context → context 包含 prompts。

- 每一層**包裹**前一層而非取代：「學會用扳手不代表丟掉螺絲起子」（arXiv 2607.00038）
- 「graph 是地圖，loop 是走路」——node 內部照樣跑 agent loop，graph 只管 node 之間的路由
- 跳過下層直接蓋上層 = 「把弱 agent 接成 org chart，你得到的是一個弱的 org」

---

## 2. 各層詳解

### 2.1 Prompt Engineering（2023–24）

- **設計物件**：單一請求的 wording、格式、CoT、few-shot examples
- **弱時症狀**：model 誤讀那一次 call 的指令
- **沒死**：loop 的每次迭代仍然需要 prompt；framework 越高階，prompt 越藏在 edge 與 tool 描述裡
- **Drill 對應**：各 agent persona 的 system prompt 與 tool 描述（`features/*/spec.md` 的 agent 定義）

### 2.2 Context Engineering（2025）

- **定義**（Karpathy）：「用恰好的資訊填滿 context window 的精緻藝術與科學」；Anthropic 正式化為「在推論期間策展並維護恰好的 token 集合」
- **設計物件**：system prompt 的位置、對話歷史保留/摘要/丟棄、RAG chunk 取捨、tool 定義暴露幾個、tool 輸出截斷、AGENTS.md 常駐注入、skills 按需載入（progressive disclosure）
- **與 prompt 的差別**：prompt 是你手寫的那部分；context 還包括檢索取回的、前幾輪留下的、tool call 回傳的——你沒有逐字重打的那些
- **弱時症狀**：model 缺所需事實，或淹死在不需要的 token 裡（context rot）
- **Drill 對應**：`ContextMode` 三種（fully-isolated / shared-project / fresh-inherits-task，見 [features/normal/spec.md](features/normal/spec.md)）；skills 的 progressive disclosure；reviewer 的 clean-context 設計是 context 工程的極致應用——只帶 claim + sources（見 [features/review/spec.md](features/review/spec.md)）

### 2.3 Harness Engineering（2026 初）

- **公式**（Trivedy）：**agent = model + harness**；harness 是「model 本身以外的每一行程式、設定與執行邏輯」
- **設計物件**：tool 執行與解析、retry、sandbox、context 組裝管線、權限閘、compaction、子 agent 派遣、tracing/觀測
- **趨勢——HaaS（harness as a service）**：從建立在 LLM API 上（拿到 completion）變成建立在 harness API 上（拿到 runtime）——Claude Agent SDK、Codex SDK、OpenAI Agents SDK 同一方向
- **Anthropic 的提醒**：「harness 的每個元件都編碼了一個『model 自己做不到』的假設」——model 變強後，scaffolding 要拆；harness 是最該持續修剪的層
- **弱時症狀**：tool call 失敗沒人發現、危險動作無人攔截
- **Drill 對應**：**雙軌決策（[decisions.md](decisions.md) D1）本質就是 harness 選擇**——DSH（Cordis plugin kernel）/ PI（extension system）是兩種 harness；skills / mcp / sandbox / setting 四個管理頁管理的全是 harness 元件

### 2.4 Loop Engineering（2026/06）

- **命名**：Boris Cherny（Anthropic Claude Code 負責人）：「我不再 prompt Claude，我寫 loop 來 prompt Claude」→ Addy Osmani 定名並定義：「**loop engineering 是把『那個 prompt agent 的你』換掉——你設計那個系統**」
- **三種 loop 要分清**（arXiv 2607.00038）：
  1. 程式 loop：普通 control flow
  2. 內在 cycle：perceive–act–observe，harness 的一部分，本來就在
  3. **Loop specification**：人設計、交給 harness 的**外部可重用 artifact**——這才是 loop engineering 的物件
- **Loop specification 五件套**：

  | 元件 | 問題 | 壞設計的症狀 |
  |---|---|---|
  | **Trigger** | 什麼啟動 run？（人 / 排程 / 事件） | 還要人工貼 prompt |
  | **Goal** | 什麼可驗證的狀態算完成？ | agent「做完了」但測試沒過 |
  | **Verification** | 怎麼真的檢查結果？ | 無限循環或過早停止 |
  | **Stopping rule** | 何时放棄、何时交还人類？ | 永不停止 |
  | **Memory** | 什麼跨迭代持久？（在磁碟，不在對話） | 重讀同檔案、重複同錯誤 |

- **驗證階梯（五級）**：從「model 自評」爬到「獨立測試套件」；Loop Library 語料（50 個真實 loop）：70% 在自治區驗證、74% 有命名終態；最弱的是自動 trigger 與持久 memory
- **命名終態（terminal states）**：`success / no-op / blocked / stalled / exhausted`——絕不把錯誤誤判為成功
- **Ralph loop**： 攔截 model 的離開意圖、把原 prompt 注入乾淨 context window 強制續跑——「最容易吞下的簡化版」，每輪乾淨、狀態走磁碟
- **Osmani 五個積木**：Automations / Worktrees / Skills / Plugins & Connectors / Sub-Agents，外加外部 Memory
- **Drill 對應**：pipeline 節點內 mini agent 的 multistep 執行；reviewer 的 sync/async 驗證 = verification；pipeline 狀態機（running/success/failed/paused）≈ terminal states；節點的 retry / acceptance gate =「做到好才往下」

### 2.5 Graph Engineering（2026/07）

#### 命名事件與「其實不新」

- 2026-07 中，OpenClaw 作者 Peter Steinberger 一則推文「還在講 loops 嗎？是不是該前進 graphs 了？」引爆，數日內定名 **graph engineering**（「Loop engineering 已死，graph engineering 萬歲」；「agents 正在從 while-loop 畢業成 org chart」）
- 但 Prefect 的 Jeremiah Lowin 更早（約 2026-06 PyData London keynote）已提出 **directed agentic graph**；LangGraph 1.0 更早於 2025-10 GA（每月 65M+ 下載）
- 誠實定位（LangChain 官方）：「graph engineering 不是新想法，是給一個已建立的做法取最新名字」——**是命名事件，不是能力事件**

#### 三個承諾（Josh Simmons 的表述）

1. **Nodes 是能力單元**：可以是 deterministic code、單次 LLM call、或**完整的 agent run**——「一個好 node 很無聊：只做一件事、可獨自測試、可整個換掉」
2. **Edges 是決策**：typed transition。**deterministic edge vs model-decided edge 必須顯式區分**——系統的智能住在 routing 裡，model-decided edges 是最會壞的地方，要 instrument
3. **State 是有 schema 的 object**，每次跨 edge 就 checkpoint——不是「context window 裡剛好躺著的東西」。「寫不出每個當下系統知道什麼，你有的只是 demo」

#### 不是 DAG

**Production agent graphs 通常不是 DAG**（LangChain 明說）：retry 失敗的 tool call、向用戶要缺失資訊、驗證後回改、等人類 approve、暫停三天再續——cycles 是 agentic 系統的本體。Prefect 直接把 DAG rebrand 成 **directed agentic graph**：每個 node 是「一個 harness 裡的 agent invocation」，且 parameterization / skills / tools / access / instructions / **連 model** 都釘在 node 層。

- **退化的單 node graph = loop engineering**：一個 node 拿全部工具自己轉到完，graph 結構什麼都沒做
- **Node-level capability pinning 的殺手應用——路徑依賴的能力閘控**：判斷「該不該退款」的 node **看不到** refund tool；判斷完成後，新 node 才拿到「鎖死單一客戶、只能用一次」的 refund tool。非破壞性工作在危險工具不存在的地方完成，破壞性工作在乾淨 harness 裡執行

#### 成本結構：authoring time vs inference time（Sangam Pandey 的框架）

| | Loop | Graph |
|---|---|---|
| 誰決定下一步 | Model，每次迭代 | 你，畫圖時一次 |
| Routing 成本 | 每次迭代付 routing token，且不可審查 | 付一次（畫圖的二十分鐘） |
| Cache | prefix 不穩（除非刻意釘住），cache miss | per-node prefix 穩定，cache 乾淨 |
| 主要失敗模式 | routing token 複利累加、跨輪推理丟失 | 需要一條你沒畫的邊 |

- 「loop 與 graph 的差別**不是拓撲**——loop 就是單 node 自環的 graph——差別是 **routing 決策的時機**」

#### 2026 真正新的兩件事

1. **Node 裝得下 full agent**：三年前 LangGraph 的 node 只是 deterministic code 或單次 LLM call；現在 agent 可靠到能整個塞進 node——「你在編排 agents，不只是 LLM calls」
2. **Authoring 變便宜**：「畫個爛圖，coding agent 把圖變成能跑的 orchestration code。**圖就是 spec**」——瓶頸從「表達 graph」移到「決定 graph 應該長怎樣」

#### 其他紀律

- **Failure edges**：「failure edges 通常佔真實 graph 的一半以上」——timeout、空回傳、malformed 各是一條邊；不先畫，coding agent 會替你爛掉地補
- **標記獨立 nodes** = 免費的平行化——loop 結構性做不到（它不知道下一個是誰）
- **Dynamic fan-out**：LangGraph `Send`——知道要 fan-out→fan-in，但不知道幾個目標（要抓幾篇 paper 跑完才知道）；靜態連線（n8n 式）做不到
- **Humans as nodes**：approval 是 edge in / edge out / 中間一個人——graph 可以停三天等人簽，不押著 context window 當人質
- **Budget in state**：token / 錢 / walltime 放進 state object、在 edge 強制執行——「停不住的系統不是自治系統，是帳單」
- **Durable execution**：Temporal / Prefect 把執行墊在下面，run 跨 crash / restart / 週末存活；checkpoint 讓失敗從「重跑全程」變「重跑那個 node」
- **互操作**：A2A / ACP——agents 是需要 edges 的 nodes 時，跨系統委派的標準化才有意義
- **既有框架**：LangGraph（StateGraph）/ AutoGen GraphFlow / Google ADK（sequential/parallel/loop workflow agents）/ Microsoft Agent Framework 1.0（2026-04 GA，AutoGen+SK 合併）

**Drill 對應**：[features/pipeline/spec.md](features/pipeline/spec.md) 就是 directed agentic graph——

| Graph 概念 | Drill pipeline |
|---|---|
| Node 光譜（fixed / model / agent steps） | 節點三層：simple llm / mini agent / code |
| Graph as artifact（可驗證、可生成） | WorkflowJSON schema（LLM 生成 + 驗證） |
| Trigger（cron / 事件 / toolcall） | cron + MCP toolcall 觸發 |
| Authoring step（圖 = spec） | **pipeline builder agent——比畫圖更激進：用講的長出 graph**，人在 DAG editor 檢視修改 |
| Checkpoint / resume | 執行狀態持久化，可 resume |
| Cycles（review 不過 → 回改） | pipeline 的 gate/retry 邊（驗證不過 → 邊指回上游節點） |

---

## 3. Graph vs Loop：什麼時候不要用 graph

**Pandey 判別法**：「能在執行前把整條流程畫在紙上 → graph；畫圖需要知道 step 3 回傳什麼 → loop。一分鐘答完，幾乎不會錯。」

**Loop territory**（graph 會在沒畫到的邊上失敗，或更糟——繞過你真正需要它注意的東西，給出自信但錯過重點的答案）：

- **Open-ended research**：下一個 query 取決於上一個找到什麼
- **Debugging**：下一個 hypothesis 取決於上一個失敗的測試
- 任何「有趣的部分是 agent 注意到你沒預期的東西」的任務

**誠實警告**：「大多數任務從不需要 graph」；「在 work 需要之前伸手拿 graph，是兩小時任務變成兩週框架專案的方式」。graph 允許退化（收成一個 node），loop 升級不上去——這是拿 graph 當預設的安全網，不是拿它當炫耀的理由。

**前車之鑑**：LangChain 自己的 deep research 最早用寫死的 LangGraph workflow 做，後來**改回 agentic core loop**（GPT Researcher 也做了同樣遷移）——planning / delegation / context management 讓它在 harness 裡湧現，比硬編進 graph 好。

**Drill 的分界線**：

> **形狀進 graph，判斷進 loop，探索留在 node 內。**

- **Chat / Orchestrator**（自由探索）= loop territory
- **Pipelines**（可重複的研究流程）= graph territory
- **蒸餾出來的 workflow 模板**（調查大家的研究用法與痛點 → 抽象共同模式 → 沉澱成模板，見 [archive/disscuss.md](archive/disscuss.md) §2.1）是「已知拓撲 + cycle 邊允許回溯」的混合形。注意：disscuss.md §2.2 的 13-stage 表是早期討論隨手記下的草稿，不是 spec

### 補充：authored graph vs emergent graph（2026-08-16 討論）

「graph」在這波討論裡**沒有嚴格定義**（LangChain 自稱 buzzword、TrueFoundry 稱 framing contested）——它是描述框架，不是標準。真正的軸是**什麼在 authoring time 被固定**：

| | Authored graph（Pipelines） | Emergent graph（Chat / omo / DSH subagents） |
|---|---|---|
| 事先固定 | 拓撲本身（nodes + edges = WorkflowJSON） | roster + 約束（agent 定義、allowed_subagents、max_depth、tools） |
| Runtime 決定 | node 內容 | 拓撲（call 誰、何時 call） |
| 圖何時可見 | 執行前 | 執行後（execution trace / Trajectory view） |
| 術語對應 | deterministic edges 為主 | model-decided edges 的極致（所有邊都是 model 決定的） |

- **oh-my-openagent 的 agents 設計早已具備 graph 的一切元素**（nodes=agents、edges=task 派遣/mailbox、shared state=tasklist、capability pinning=per-agent model/tools、depth guard=max_depth）——只是拓撲是 runtime 湧現的，不是事先畫的
- 兩端不是「有沒有 graph」，是同一條 authoring 光譜的兩端：Drill 的 normal ↔ pipeline 正好各據一端

---

## 4. 術語速查表

| 術語 | 一句話 |
|---|---|
| prompt engineering | 設計單一請求的措辭 |
| context engineering | 設計 model 每次 call 看到的全部 token |
| harness engineering | 設計 model 以外的執行系統（tools / sandbox / retry / 權限 / 觀測） |
| loop engineering | 設計重複觸發、驗證、停止 agent 的系統 |
| graph engineering | 設計多 agent 的 nodes / edges / shared state 拓撲 |
| agent = model + harness | Trivedy 公式：harness 是 model 以外的一切 |
| HaaS（harness as a service） | 從 LLM API 走向 harness API / runtime（Agent SDK 們） |
| loop specification | trigger + goal + verification + stopping rule + memory 的外部 artifact |
| terminal states | success / no-op / blocked / stalled / exhausted |
| verification ladder | 驗證的五級階梯：model 自評 → 獨立測試套件 |
| Ralph loop | 攔截離開、重注入 prompt 的自續 loop（loop engineering 最簡版） |
| directed agentic graph | 允許 cycles 的 agent 編排圖（DAG 的 rebrand） |
| node | 能力單元：deterministic code / 單次 LLM call / full agent |
| edge | typed transition：deterministic 或 model-decided |
| failure edge | timeout / 空回傳 / malformed 的去處——真實 graph 的一半以上 |
| fan-out / fan-in | 平行發散 / 收斂 join |
| dynamic fan-out | 執行時才決定目標數量的發散（LangGraph `Send`） |
| shared state | 沿 edge 流動、有 schema、每跨 edge checkpoint 的 object |
| node-level capability pinning | model / tools / skills / access 釘在 node 層 |
| capability gating | 危險 tool 只存在於特定 path 上的 node（路徑依賴能力閘控） |
| checkpoint | 跨 edge 的狀態快照——失敗從「重跑全程」變「重跑 node」 |
| durable execution | 跨 crash / restart 存活的執行（Temporal / Prefect） |
| authoring time routing | 畫圖時決定路由（付一次）vs inference time routing（每次迭代付） |
| humans as nodes | approval 設計成 edge in / edge out / 中間一個人 |
| budget in state | token / 錢 / walltime 進 state、在 edge 強制執行 |
| A2A / ACP | 跨系統 agent 委派的互操作協議 |

---

## 5. Drill 對應地圖

| 概念層 | Drill 位置 |
|---|---|
| Prompt | 各 agent persona 的 system prompt、tool 描述（`features/*/spec.md`） |
| Context | `ContextMode` 三種、skills progressive disclosure、reviewer clean-context |
| Harness | DSH（Cordis）/ PI（extensions）雙軌（[decisions.md](decisions.md) D1）；skills / mcp / sandbox / setting 頁 |
| Loop | pipeline 節點內 mini agent、reviewer 驗證、pipeline 狀態機、節點 retry/gate |
| Graph | Pipelines：WorkflowJSON、節點三層、cron + MCP trigger、builder agent authoring |

---

## 6. 出處與延伸閱讀

**Graph engineering（2026/07 命名事件與回應）：**

- [LangChain — 3 Years of Graph Engineering with LangGraph](https://www.langchain.com/blog/3-years-of-graph-engineering-with-langgraph)（「不是 DAG」「Send」「node 裝得下 full agent」）
- [Prefect — Loops vs. Graphs](https://www.prefect.io/blog/loops-vs-graphs)（2026-07-22；Lowin 的 directed agentic graph、node-level settings、能力閘控）
- [Sangam Pandey — When an Agent Loop Should Be a Graph](https://sangampandey.info/blog/graph-engineering-agent-loops-to-graphs)（2026-07-27；authoring vs inference time、判別法、failure edges）
- [Josh C. Simmons — We Are Entering the Graph Engineering Phase](https://www.drjoshcsimmons.com/writing/we-are-entering-the-graph-engineering-phase)（三承諾；「ready set = 1 的 scheduler」引 arXiv 2604.11378）
- [DZone — The Layer After Loop Engineering](https://dzone.com/articles/understanding-graph-engineering)（2026-08-14；「graph 是地圖，loop 是走路」）
- [Louis Bouchard — Graph Engineering Explained](https://www.louisbouchard.ai/graph-engineering-explained/)（2026-07-22）
- [AI Builder Club — Graph Engineering Guide 2026](https://www.aibuilderclub.com/blog/graph-engineering-guide-2026)（2026-07-20；「大多數任務不需要 graph」）
- [TrueFoundry — Graph Engineering Enterprise Guide](https://www.truefoundry.com/blog/graph-engineering-enterprise-guide)（命名事件系譜）

**Loop engineering：**

- [arXiv 2607.00038 — Loop Engineering position paper](https://arxiv.org/html/2607.00038)（loop specification 解剖、五級驗證階梯、terminal states、Loop Library 語料）
- [arXiv 2604.11378 — From Agent Loops to Structured Graphs]（「agent loop 是 ready set 恰為 1 的 scheduler」）

**Harness / 階梯總覽：**

- [Addy Osmani — Agent Harness Engineering（O'Reilly, 2026-05-15）](https://www.oreilly.com/radar/agent-harness-engineering/)（harness 定義、HaaS、Ralph loop）
- [codecentric — Loop, Harness, Context Engineering Explained](https://www.codecentric.de/en/knowledge-hub/blog/loop-harness-context-engineering-explained)（2026-07-05；Cherny / Osmani 命名時間線）
- [FutureAGI — Prompt, Context, Harness, Loop: 4 Agent Layers](https://futureagi.com/blog/loop-engineering/prompt-context-harness-loop-layers/)（2026-07-26；階梯年表）
- [FutureAGI — Loop vs Harness Engineering](https://futureagi.com/blog/loop-engineering/loop-engineering-vs-harness-engineering/)（2026-07-21）
- [explainx.ai — The 2026 Engineering Stack](https://explainx.ai/blog/context-prompt-loop-harness-engineering-stack-2026)（2026-06-29；nesting rule）
- [Daniel Liden — Harness Engineering](https://danliden.com/posts/20260228-harness-engineering.html)（2026-02-28；harness 極簡自建實錄）
