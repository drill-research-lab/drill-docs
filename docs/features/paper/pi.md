# Paper Mode — Track A（PI-based）設計

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/pi.md](../../tracks/pi.md)。
> 原則：最大化複用 PI 生態系（[reference/pi-ecosystem.md](../../reference/pi-ecosystem.md)），gap 才自建。

## 撰寫工具鏈（自 spec 移入；plugin 詳情 → [reference/pi-ecosystem.md](../../reference/pi-ecosystem.md) 學術節）

| 功能 | 工具 | 備註 |
|---|---|---|
| **文獻搜尋** | hinsencamp/pi-research-agent（SS+arXiv+OpenAlex，免 key）| 搜尋+摘要+ranking |
| **論文生成** | Aspis0/pi-paper-lab（anti-AI rewrite + Vancouver citations + .docx Word-native fields；backends：CrossRef/Serper/Exa/OpenAlex/Europe PMC）| DeepWiki 已驗證 |
| **引用驗證** | pi-bib（CrossRef + Semantic Scholar 檢查 BibTeX、duplicate 偵測）| DOI-first——`verify_citations` tool 的現成底座 |
| **全文抓取** | pi-arxivist（pandoc WASM，math 保留）/ @wienerberliner/pi-arxiv | LaTeX source → MD |
| **Zotero** | paper-pilot **MCP**（6 DB + OA PDF + evidence extraction + Zotero sync）/ academic-agent | 文獻庫管理；經 [mcp/pi.md](../mcp/pi.md) 的 adapter 接入 |
| **完整學術套件** | portos-wang/pi-extensions（39-agent：Deep Research 13 / Paper 12 / Reviewer 7 / Pipeline 10）| 未經 DeepWiki 驗證；整包參考 |

## 報告產生器的模板系統（實作）

- 模板 = LaTeX/MD 模板目錄（如 `nstc-proposal/`）+ 對應 skill（`nstc-proposal` SKILL.md）
- writer agent 載入 template + skill → 產出草稿 → compile → PDF
- 首個模板：國科會計畫書（見 spec 開放問題：官方模板待確認）

## 自建 gap 清單

1. `write_section` / `rewrite_section` / `add_citations` / `compile_latex` / `format_references` tools——以 pi extension 包裝上述 plugins（pi-paper-lab/pi-bib 已覆蓋大半，缺的補齊）
2. **LaTeX compile service**——tectonic（單一二進位）或 latexmk in Docker；容器隔離 + PDF cache + incremental（Overleaf CLSI 模式，見 [reference/domains.md](../../reference/domains.md) §5）
3. 編輯器前端（CodeMirror 6 + PDF 並排；協同編輯 = Yjs，時程見 spec）
4. 模板倉（NeurIPS / ICML / IMRAD / 國科會 / 技術報告…）
