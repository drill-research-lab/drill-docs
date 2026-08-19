# Lab — Track B（DSH-based）註記

> 對應 [spec.md](spec.md)。Track 總覽：[tracks/dsh.md](../../tracks/dsh.md)。
> 深度調查：[reference/dsh.md](../../reference/dsh.md)。

## seam / plugin 對應

| spec 需求 | DSH 對應 |
|---|---|
| 沙盒 | `ctx.fs` seam（`fs-local` → 可換 sandboxed provider）+ DSH sandbox package（`packages/sandbox/sandbox/`）；`tool-bash` / `tool-fs` 是 standard preset 內建 |
| coding agent 工具面 | `standard` preset 的 23 tools（read/write/edit/bash…）大多覆蓋；`code` preset 的 Code Mode 可作程式密集強化 |
| 回溯機制 | 待查（session log + git 層）|
| Code Mode | `code` preset：model 寫 TS 打 generated SDK（五 round trip 變一個）——researcher 程式密集任務候選 |
