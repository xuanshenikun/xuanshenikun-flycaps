# Flyme 实况通知协议（实机验证）

> 基线：MEIZU 21 / Flyme 12 / Android 16（API 36）· 2026-11-21 实测
> 验证方法：自建测试 APK，逐字段试 + `dumpsys notification` 对照 + 截图确认

## 1. 触发方式

普通 App 用 `NotificationManager.notify()` 发一条通知，在 `Bundle extras` 里塞
`notification.live.*` 字段即可。**shell 发不了**：`cmd notification post` 不支持 extras（实测）。

## 2. 准入门槛

```bash
# 白名单（逗号分隔，root 写入）
settings get system mz_app_support_livenotification_pkgs
settings put system mz_app_support_livenotification_pkgs "<现有值>,com.example.app"
```

- SystemUI 显示门禁（反编译确认）：`sbn.isLive() && sbn.getCapsuleStatus() == 1`
  → **`notification.live.capsuleStatus` 必须为 1**，否则不显示
- NMS 接口：`getNotificationLiveAllowForPackage` / `setNotificationLiveAllowForPackage`
- 通知使用权（NLS）是另一个独立授权：
  `settings get secure enabled_notification_listeners`

## 3. 胶囊字段（全部在 Bundle extra `notification.live.capsule` 内，键名要带全前缀）

| 键 | 类型 | 说明 |
|---|---|---|
| `notification.live.capsuleContent` | String | 胶囊文字（歌名/标题） |
| `notification.live.capsuleIcon` | Icon | 胶囊图标（实测放 App 图标） |
| `notification.live.capsuleType` | int | 1 = 文本型 |
| `notification.live.capsuleStatus` | int | **必须 = 1** |
| `notification.live.capsuleBgColor` | int | 胶囊背景色（实测用 App 图标平均色 → 自适应效果） |
| `notification.live.capsuleContentColor` | int | 文字色（白 0xFFFFFFFF） |
| `notification.live.capsulePause` | boolean | 暂停态 |
| `notification.live.capsulePauseTime` | String | 暂停时长文案 |
| `notification.live.capsuleCountDown` | boolean | 倒计时开关 |

> ⚠️ 键名虽然带前缀，但值是**放在 capsule Bundle 内部**的，不是 Notification 顶层。

## 4. 实况卡片字段（通知顶层 extras）

| 键 | 类型 | 实测效果 |
|---|---|---|
| `notification.live.type` | int | 1 |
| `notification.live.event` | String | "MUSIC" |
| `notification.live.pinned` | boolean | true（钉状态栏） |
| `notification.live.titleOverlay` | String | 卡片标题 |
| `notification.live.contentOverlay` | String | 卡片副标题 |
| `notification.live.iconOverlay` | Icon | **实况卡片左侧图标**（实测放封面 Bitmap 即显示封面） |
| `notification.live.feature.extendImage` | ArrayList<Icon> | 卡片按钮图标 |
| `notification.live.feature.chronometerBase` | long | 进度基准（elapsedRealtime ms，不是 epoch） |
| `notification.live.feature.chronometerPause` | boolean | 进度暂停 |
| `notification.live.feature.progressHide` | boolean | 隐藏进度条 |
| `notification.live.operation` | int | 1 |
| `notification.live.app.pkg.delegated` | String | 委托包名（见第 6 节） |
| `notification.live.capsule.content.remote.view` | RemoteViews | 卡片自定义视图 |
| `notification.live.capsule.click_launch_pending_intent` | PendingIntent | 点胶囊跳转 |

## 5. 按钮映射（重要坑）

- 实况卡片的按钮 = `extendImage`，点击触发通知的 **`actions[i]`**（同索引）
- 但**横向显示顺序是反的**：`extendImage[2]` 在最左、`extendImage[0]` 在最右
- 实测：想显示「⏮ ⏸ ⏭」，extendImage 要按 `[⏭, ⏸, ⏮]` 顺序放
- 每个按钮图标必须与对应 action 的语义一致，否则会"图标是上一首、点了却是下一首"

## 6. 委托机制

```java
ex.putString("notification.live.app.pkg.delegated", targetPkg);
```

- SystemUI 的 `getLiveNotificationEntryAppPkg` 读它
- 效果：实况通知被当作目标 App 的（如酷狗/B站），**关闭目标 App 时胶囊的收起动画归属正确**
- 不设的话动画归属到你的 App

## 7. 已验证的坑

| 坑 | 现象 | 结论 |
|---|---|---|
| 加 `MediaStyle` | 胶囊消失 | 通知被归类为"媒体通知"进媒体卡片轮播，`getRow()` 为空 → 不显示胶囊 |
| 想同时要胶囊+系统媒体卡片 | 不可能 | MediaStyle 与胶囊互斥（同一通知），要两个就得发两条通知 |
| RemoteViews 塞大图 | 卡片图标不更新 | 有 parcel 大小限制，Bitmap 缩到 96×96 |
| 只改 art 不触发更新 | 封面不出现 | 更新条件要包含"封面从无到有" |
| 每秒 notify | 卡顿/闪烁 | 只在 标题/歌手/状态/封面 变化时更新，进度用 chronometer 或干脆不追 |
| 通知 id 复用 FGS | 占位通知覆盖真实内容 | `startForeground()` 二次调用会重置，必须传真实通知 |
