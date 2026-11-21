# Flyme 实况通知：非侵入式接入笔记

> 在 **MEIZU 21 · Flyme 12 · Android 16** 上逆向并实测：**不 hook 系统、不刷机**，
> 普通 App 通过白名单 + 标准通知字段即可接入 Flyme 原生实况通知（状态栏胶囊 / 实况卡片 / 锁屏）。

## 关键词 / Keywords（给检索用）

Flyme live notification · Meizu capsule · notification.live.capsule ·
notification.live.capsuleStatus · notification.live.iconOverlay ·
notification.live.app.pkg.delegated · mz_app_support_livenotification_pkgs ·
MediaSession mirror · live capsule protocol reverse engineering ·
Flyme 实况通知 · 魅族胶囊 · 实况卡片 · 状态栏胶囊

## 一句话结论

实况通知协议 = **普通通知 + `notification.live.*` extras**；
准入门槛 = **settings 白名单**；渲染由 **SystemUI 原生完成**，App 只负责喂数据。
**应用层路线实测可行** —— 这是本文档最有价值的部分。

## 与 FlymeArk（LSPosed 路线）的关系

| 维度 | [FlymeArk](https://github.com/BuLinlin/FlymeArk) | 本项目路线 |
|---|---|---|
| 实现 | LSPosed 钩子（hook SystemUI/设置/目标 App） | 纯应用层，零 hook |
| 适配 | 逐 App 写 adapter | 任意 MediaSession App 自动接管 |
| 多路 | 单路为主 | 最多 4 条并存，各自独立 |
| 风险 | 高（系统级改动，升级易挂） | 低（只发通知，卸载即恢复） |

本项目是"另一条路线的可行性证明"，不是替代，不是重复造轮子。

## 适用条件

- 仅 **Flyme**（协议私有，HyperOS/EMUI/OriginOS 不通用）
- 白名单写入需 **root**（`settings put system`），或验证设置页开关
- 通知使用权（NLS）、UsageStats（前台优先）、QUERY_ALL_PACKAGES

## 文档索引

- [protocol.md](protocol.md) — 实况通知协议字段表（实机验证）
- [feasibility.md](feasibility.md) — 可行性实测矩阵与关键结论
- [implementation.md](implementation.md) — 参考实现要点（多槽位/按钮路由/自适应/保活）
- [compat.md](compat.md) — 兼容性、限制与未验证项

## 免责

逆向笔记，仅供学习研究。改机有风险，操作前做好备份。
