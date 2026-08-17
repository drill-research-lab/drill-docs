# Review Mode — Track A（PI-based）設計

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/pi.md](../../tracks/pi.md)。
> 原則：最大化複用 PI 生態系（[reference/pi-ecosystem.md](../../reference/pi-ecosystem.md)），gap 才自建。

## 複用組合

| 需求 | 複用 |
|---|---|
| clean-context 驗證 session | **pi-subagents** 的 `fresh` context mode（`subagent()` 起獨立 child）——reviewer 的核心形態現成 |
| 多源交叉驗證的搜尋面 | **pi-web-access**（web_search / fetch_content；provider fallback chain）|
| 引用驗證 | **pi-bib**（CrossRef + Semantic Scholar BibTeX 檢查、duplicate 偵測）——`check_citations` 底座 |
| 學術 claim 查證 | **hinsencamp/pi-research-agent**（SS + arXiv + OpenAlex）|
| 多視角 review 參考 | pi-subagents 的 reviewer/oracle builtin agents；portos-wang 7-agent reviewer 套件（未經 DeepWiki 驗證）|
| 深度調查模式 | **@quintinshaw/pi-dynamic-workflows** 的 `/deep-research`（web search + cross-checked claims 的 fan-out workflow）|

## 自建 gap 清單

1. `verify_claims` / `cross_check` / `review_document` / `suggest_revisions` tools（組合上述 plugins 為 reviewer 的工具面）
2. 驗證分級狀態機（VERIFIED / PARTIALLY SUPPORTED / CONTRADICTED / UNVERIFIABLE）——判斷邏輯自建
3. claim-level 標記 UI（綠/黃/紅）＋ claim↔source 對照視圖（前端）
4. 七視角 rubric 的七個輕量 agent 定義（MD + frontmatter）
