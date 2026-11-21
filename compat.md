# 兼容性、限制与未验证项

## 已验证环境

| 项 | 值 |
|---|---|
| 机型 | MEIZU 21 |
| 系统 | Flyme 12 / Android 16 (API 36) |
| root | KernelSU 临时 root（df.root + Shizuku） |
| 状态 | 胶囊/卡片/按钮/多路/委托/自适应全部跑通 |

## 限制

1. **仅 Flyme**：协议私有，HyperOS 焦点通知、鸿蒙实况窗、OriginOS 原子通知各有自己的 API，完全不通用
2. **白名单需 root**：`settings put system` 需要 root/系统权限。
   未验证：设置 → 通知与状态栏 → 实况通知 的开关能否直接加第三方 App（若能，则无需 root）
3. **MediaStyle 互斥**：想同时要"系统媒体卡片"和"实况胶囊"，只能发两条通知
4. **非媒体 App**：无 MediaSession，必须逐家解析通知文本
5. **不同 Flyme 版本**：12.5/12.7 字段可能变化，需按固件验证
6. **视频类会话名因 App 而异**：B站是 `HeadsetControlPlayAdapter.Session`，抖音等需实测

## 未验证项

- 锁屏 / AOD 上胶囊的显示（只验证了桌面状态栏）
- 设置页开关是否能替代 root 写白名单
- Flyme 后续版本对 `notification.live.*` 字段的兼容性
- 更多 App 的会话暴露情况（抖音、淘宝直播、播客类）
