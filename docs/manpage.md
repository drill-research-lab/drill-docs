# manpage 

左邊側邊

page:

normal mode
wiki mode
pipeline mode
research mode
paper mode
review mode
skills
mcp
setting page
system status


## normal mode

normal chat mode

like chatgpt/gemini web
但我想讓他是 hermes agent

可以 call 其他 agent as subagent 作任何事情
像是 call librarian 某個知識庫的內容
或是 call reviewer 驗證事實

📄 詳細設計 → [`design/normal.md`](design/normal.md)

## wiki mode

類似 notebookLM 與 llm-wiki
使用者可以上傳任意文件，llm 會自動協助與整理，可能會使用 anydoc 或是 markitdown 或是 ocr
RAG ?

AGENT: librarian

📄 詳細設計 → [`design/wiki.md`](design/wiki.md)

## pipeline mode

可以建立 pipeline 與 cornjob 
就像是 n8n 在做的事情
使用者或是 pipeline builder 可以設定 cornjob 定期執行或是被 toolcall 觸發 (as mcp)
每個節點可以是程式碼/agent/llm
建立完之後可以被 agent 呼叫 (like mcp) 或是 cronjob 觸發

AGENT: 可以 mutistep 執行（設定 prompt/skills/tools），完成後傳給後續節點
llm: single llm ask 然後傳給後續節點

AGENT: pipeline builder / mini agent / simple llm

📄 詳細設計 → [`design/pipeline.md`](design/pipeline.md)

## research mode
可以建立實驗或是 poc
可以編輯程式碼
llm 生成的文件或程式碼可以讓用戶點開在右邊查看，框起來 suggest fix 或編輯
也可以編輯任意程式碼
就像 coding agent (claude/codex/opencode)
用起來會像是 vscode + copilot

AGENT: researcher

📄 詳細設計 → [`design/research.md`](design/research.md)

## paper mode
跟 agents 一起建立 paper
md or latex
自帶 latex builder / preview / pdf viewer
就像 overleaf 一樣

AGENT: writer

📄 詳細設計 → [`design/paper.md`](design/paper.md)

## review mode

review 產生的結果
或是 review 其他東西

也可以被其他 agent 呼叫 ，在無污染上下文的情況驗證事實，anti 幻覺

📄 詳細設計 → [`design/review.md`](design/review.md)

## skills 
add import edit skills files

📄 詳細設計 → [`design/skills.md`](design/skills.md)

## mcp
add import edit mcps 

📄 詳細設計 → [`design/mcp.md`](design/mcp.md)

## setting page

just any setting

📄 詳細設計 → [`design/setting.md`](design/setting.md)

## system status

system status
backend
frontend
llm
network
storage..etc

📄 詳細設計 → [`design/system-status.md`](design/system-status.md)

---

## 文件索引

| 文件 | 內容 |
|---|---|
| [reference.md](reference.md) | 所有參考專案總覽（OSS/CLOSED/MAIN 標記、深度調查、命名陷阱） |
| [DSH.md](DSH.md) | DeepSeek Harness 深度設計調查（15 面向 + 設計決策輸入，commit 級驗證） |
| [concepts.md](concepts.md) | Agent 工程術語階梯：prompt / context / harness / loop / graph engineering（2026-08 基準） |
| [decisions.md](decisions.md) | 專案決策紀錄（雙軌策略、自進化、storage 定義、部署、分工） |
| [design/](design/) | 每個 mode 的詳細設計 |
| [disscuss.md](disscuss.md) | ⚠️ 已棄用——早期 Spark 階段討論，僅存歷史 |
| [art.md](art.md) | 品牌 / 命名（鑽頭、天元突破） |
