# 参考实现要点

> 完整代码思路，不是可直接编译的工程（演示原型在 /root/livenotify，未整理发布）。

## 服务结构

```
MirrorService (FGS, START_STICKY)
 ├─ 轮询 MediaSessionManager.getActiveSessions()  (1.2s)
 ├─ 槽位分配（最多 4 个会话，每槽一个包，稳定保留）
 ├─ 每槽：读元数据/封面/状态 → 组通知（id 2..5）
 └─ 广播接收（按钮）→ 按 slot 路由到对应 MediaController
LiveNls (NotificationListenerService)
 └─ 仅用于获得 NLS 权限（getActiveSessions 需要）
BootReceiver
 └─ BOOT_COMPLETED → startForegroundService
```

## 关键实现

- **槽位**：`Slot[] slots = new Slot[4]`；会话存在（PLAYING/PAUSED/BUFFERING）就保留，STOPPED/NONE 清空
- **按钮路由**：广播 `MCMD` + `slot` extra，`forward(slot, cmd)` 调对应 controller 的 TransportControls
- **自适应配色**：`capsuleBgColor = 图标 Bitmap 平均色`（遍历像素取 RGB 均值，跳过透明）
- **封面**：`METADATA_KEY_ALBUM_ART → ART → DISPLAY_ICON → URI(ContentResolver)` 逐级兜底
- **变化检测**：`title|artist|playing|hasArt|duration` 签名，变化才 notify（避免卡顿）
- **取消**：清槽时保留 Slot 对象，用 `posted` 标记判断是否 `nm.cancel()`
- **startForeground**：有内容用真实通知，否则占位（二次调用会覆盖）

## 保活

```bash
# 开机自启：BOOT_COMPLETED 接收器
# 电池白名单
dumpsys deviceidle whitelist +com.example.app
appops set com.example.app RUN_ANY_IN_BACKGROUND allow
# 应用内按钮：ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS
```

## 权限清单

- `BIND_NOTIFICATION_LISTENER_SERVICE`（NLS）
- `POST_NOTIFICATIONS`
- `FOREGROUND_SERVICE`（targetSdk 33 免类型声明）
- `RECEIVE_BOOT_COMPLETED` / `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS`
- `QUERY_ALL_PACKAGES`（读其他 App 图标）
- `PACKAGE_USAGE_STATS`（前台优先，appops 授予）
