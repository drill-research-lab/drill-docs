# 老記(neokent)追加需求原文（2026-08-19）

> 狀態：逐項討論中。`原文` 區保持原樣；確認後的結論另記於文件末尾，並同步到正式 spec。

## 原文
然後他希望多這些功能

```text

翻譯頁面：左側 UI 有獨立入口，不需要進聊天。上傳檔案 → 翻譯 → 直接預覽結果 → 可下載。它本質上是 file-in / translated-file-out 的工具。
定時搜尋：設定關鍵字、搜尋來源、執行頻率，後端建立排程，定期搜尋相關資料集。這是 background scheduled job，不需要使用者一直開著頁面。
PDF 預覽：在聊天裡，如果 Agent 產生或修改了一個 PDF，訊息上直接有「預覽」和「下載」。點預覽就在 DSH 裡看，不需要先下載到本機再開。
論文預覽：自己上傳的論文，或定時搜尋找到的論文/檔案，都能在左側 UI 直接選取並預覽。也就是「資料庫/論文庫 + viewer」。
工作區：主要是 UI 組織層。把翻譯、論文、資料集、筆記等能力集中到一個 Workspace 區域，而不是全部塞在聊天頁。
共用資料集：過往以搜尋的部分可以直接取用而不需要重新跑。
筆記本：重點不是自由寫筆記，而是整理每個 conversation 的重點。例如一個 session 結束後，可以得到這次討論的結論、重要資料、下一步等，之後從 Notebook 回頭看。


```
上面要做的我還沒看哪些已經支援了 先列出他剛剛希望的部分
感覺差不多我在 Docs 內都有 cover 到。

## 討論順序

1. 翻譯頁面
2. 定時搜尋
3. PDF 預覽
4. 論文預覽
5. 工作區
6. 共用資料集
7. 筆記本

## 已確認的討論結果

### 3 / 4. PDF 預覽與論文預覽

- Chat、Writer、Wiki／論文庫共用同一套 PDF viewer，但三者的使用流程不同
- **Chat**：Agent 產生或修改 PDF 後，訊息直接提供預覽與下載；見 [normal spec](../features/normal/spec.md)
- **Writer**：方向是與 LLM 協作的 Overleaf，具自動版控；LLM 一輪回應若修改論文，結束後重新編譯一次並刷新預覽；顯示 LaTeX LSP 與編譯錯誤；見 [paper spec](../features/paper/spec.md)
- **Wiki／論文庫**：自行上傳與 Scheduled Search 取得的論文進入同一資料庫；保存原始 PDF，並以 anydoc / markitdown 候選流程轉成 Markdown；原始 PDF 用於預覽，Markdown 用於整理、搜尋與 LLM；見 [wiki spec](../features/wiki/spec.md)
- 此階段只定義產品行為；Viewer 如何引用或取得檔案尚未決定，不預設檔案引用方式或其他實作 schema

### 6. 共用資料集 → Pipeline outputs reuse

- 此需求暫不限定為表格型 dataset；核心是 Pipeline 的來源與結果可以保存並供後續研究重用
- Pipeline 未配置 output destination 時，sources 與 results 預設保存到目前 Project
- 使用者可在執行後手動將內容加入 Wiki，也可在 Pipeline / template 中配置自動流入指定 Wiki
- 即使自動流入 Wiki，Project 仍保留該次 pipeline run 與結果紀錄
- Source Markdown 與 Result Markdown 都可進 Wiki，但必須保留「原始來源」與「模型綜整結果」的差異
- Normal Chat 透過 librarian subagent 查詢 Wiki 內容並帶回 conversation 使用
- 詳細規則見 [pipeline spec](../features/pipeline/spec.md)、[wiki spec](../features/wiki/spec.md)與 [Workspace / Project 所有權模型](../workspace-project-ownership.md)

### 7. 筆記本 → Conversation Note / Notebook

- 不使用「session 結束後」作為觸發條件，因為 Conversation 沒有可靠的自然結束事件
- 每個 Conversation 最多自帶一份 Markdown **Conversation Note**，整理目前討論供使用者與其他 agent 閱讀
- 尚未建立時可「產生 Note」；建立後可「更新 Note」
- 使用者可請 LLM 修正 Note；必要時可直接修改，或選擇重新產生
- **Notebook** 是 Conversation Notes 的集中瀏覽入口，不是第二套 Wiki，也不是自由筆記工具
- Project Conversation 的 Note 歸 Project；Workspace scratch Conversation 的 Note 歸 Workspace
- LLM 更新 Note 的工具模型、局部修改方式與版本處理暫不深入
- 完整規格見 [Conversation Note / Notebook](../conversation-note.md)
