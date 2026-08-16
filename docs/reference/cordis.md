# Cordis — DSH 的 plugin kernel

> Koishi 團隊的 meta-framework；DSH 的核心。paper 完整深度分析。
> 相關：[dsh.md](dsh.md)（DSH 調查）、[README.md](README.md)（總覽）。

---

| 項目 | 內容 |
|---|---|
| **Repo** | [github.com/cordiverse/cordis](https://github.com/cordiverse/cordis)（Koishi 團隊的 meta-framework） |
| **Paper** | [github.com/cordiverse/paper](https://github.com/cordiverse/paper) — *A Programming Paradigm for Spatiotemporal Composability*（北大 + DeepSeek，2026-08-13，preprint） |
| **核心概念** | **Temporal composability**（卸載 = 完全逆轉副作用）+ **Spatial composability**（宣告 + 反應式管理依賴） |
| **機制** | **Revertible effects**（每個 context transformation 帶 inverse，runtime 追蹤，`recover()` 逆序執行回初始態）+ **Reactive coeffects**（component 宣告依賴規格，context 變動時自動 notify activating/deactivating/neutral） |
| **形式化** | Calculus of dynamic composition；metatheory：Preservation / Temporal Composability / Spatial Composability / Progress / Confluence |
| **驗證** | Koishi chatbot：4 年、4000+ 社群 plugin、TypeScript |
| **設計啟示** | 比 PI extensions 更強：數學保證「裝得進去 = 拆得乾淨」；對 multi-agent（agent 互相 call）特別重要——卸載 agent 時 dependents 自動暫停 |

### 深度調查（完整讀完 paper 後）

**形式模型核心：**
- **Effect context** `∂Γ = Γ × (Γ→Γ)`：pair `(γ, φ)`，γ = 目前狀態，φ = accumulator（所有已執行 effect 的 inverse 合成）
- `track_Γ(f,g) = (γ,φ) ↦ (f(γ), φ∘g)` — 套用 forward map + 把 inverse 合成進 accumulator
- `recover_Γ = (γ,φ) ↦ (φ(γ), id_Γ)` — 套用 accumulator 回到初始態
- **Witnessed effect function** `𝔈*_Γ`：每個 effect 在套用點回傳 inverse，並帶 proof `g(δ)=γ`（inverse 真的還原）；Theorem 11：witnessing 在合成下封閉
- **Component**（formal）= triple `(d, p, e)`：d = 依賴規格（需要的 keys）、p = provision（提供的 keys）、e = witnessed effect function
- **單一來源紀律**：no two fibers 可提供重疊的 key → 一個 provider 一次只有一個 fiber

**5 個 metatheorem（保證什麼）：**
1. **Preservation**：10 條規則都保持 registry well-formed
2. **Temporal composability**：Recovery exactness — 套用 fiber 的 accumulator 只撤銷它自己的貢獻，不碰別人
3. **Spatial composability**：Ordering — provider 活得比 consumer 久；Resolution coherence — 每個 iteration 對固定 resolution 執行
4. **Progress**：無 deadlock（非 quiescent 必有規則可套）+ 終止（`S(n) ≤ (K+4)(V(n)+1)`）
5. **Confluence**：quiescent state 是 config 的純函數 — 任何 schedule 收斂到同一個「靜態組裝」normal form

**實作（Cordis 4）：**
- `ctx.effect(callback)` = **唯一 mutation primitive**，實作 effect iterator，回傳 `dispose()` closure
- ⚠️ 命名校正（DeepWiki 實證 2026-08-16）：paper（v4）寫 `ctx.use(component, config)`，但 **repo 實際 API 是 `ctx.plugin(plugin, config)`**——套用 plugin 時建立 Fiber 管理其 lifecycle；概念相同、名稱不同
- `ctx.get/set`（keyed, typed）；`ctx.isolate(key, realm)` + `ctx.intercept(key, metadata)` = derived child context（無需 inverse）
- **Proxy access control**：`ctx[key]` 對照 fiber 的 committed view；undeclared → `UNDECLARED_ACCESS`；capability-style 的 load-time review
- **Loader**：entry = `(id, url, isolate, intercept, config, disabled)`；incremental reconciliation；`@cordisjs/group` / `@cordisjs/include` 是普通 component
- **HMR**：3 階段（import graph classification / stale detection / transactional reload with cache backup-rollback）；不需要 Webpack/Vite 的 annotation；ESM 無 public cache-eviction API 是 caveat
- **Koishi** 跑 Cordis v3；paper 呈現 v4（核心模型共享，semantics 精煉 + loader 重設計）

**誠實的限制（paper 自己承認）：**
1. **Inverse witness 沒有 runtime 檢查** — inverse 正確性是 component 作者的義務，runtime 不驗證（寫錯就 silent leak）
2. Recovery 是 **observational equivalence ≃**，不是 literal state equality（heap layout / names 不還原）
3. 保證**有前提**：所有共享位置綁在 commutative key、provision totality、`≺` acyclic、finite names
4. **Confluence 排除 failure**（failure 是真正 divergence source）
5. **Emissions（寫出/送出）無法 revert** — 只能 withheld 或 compensated（後者破壞 metatheory 的 commutation）
6. **Own in-memory state 不 survive reload**（forward state migration 是 future work）
7. **無 quantitative validation** — 單一 ecosystem（Koishi）+ 單一語言（TS）；agent harness 驗證明說 "compelling future direction"
8. 沒 benchmark — Proxy trap / per-effect closure / notification fan-out 的 overhead 未量化

**對 Drill 的決策輸入（§7 完整分析）：**
- **Clear win 的場景**：tool/subagent 真的需要在 runtime 頻繁 load/unload/replace + 需要 teardown 保證 + 需要 audit——這是 paper 明點的 motivating case（self-evolving agent harness）
- **Wash 的場景**：plumbing（message passing、session state 可以留在 context 外）
- **Overkill 的場景**：工具只在 boot 時載一次，之後不動——Cordis 的紀律（一切過 `ctx.effect`、inverse、disjoint provision）純成本
- **採用路徑**：先只拿 core API（`ctx.effect`/`ctx.get/set`/`ctx.use`）；每個 tool/subagent/session 當 component（inject/provide）；per-subagent 用 isolation realm；用 Thm 73 當 test oracle（quiescent state = static config）；bad inverse 用「dispose 冪等」test harness 抓
