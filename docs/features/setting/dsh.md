# Setting 頁 — Track B（DSH-based）註記

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/dsh.md](../../tracks/dsh.md)。
> 深度調查：[reference/dsh.md](../../reference/dsh.md)。

## seam / plugin 對應

| spec 需求 | DSH 對應 |
|---|---|
| 設定持久化 | `settings-file` plugin（dsh-base 內建）|
| models 設定 | `agent-default-model`（provider `deepseek-official` / `deepseek-v4-flash`）+ `ui-model-selection` |
| 設定 UI | `ui-settings*` 系列（general/models/plugins/plugin-inventory）——現成可擴充 |
| 權限 | `ui-permission-presets` + approval policy（dsh-base 內建）|
| 用戶/多租戶 | fork 上的用戶隔離工作（阿柏）——見 [tracks/dsh.md](../../tracks/dsh.md) |
