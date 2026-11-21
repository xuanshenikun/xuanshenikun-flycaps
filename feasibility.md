# 可行性实测矩阵

> 核心问题：**不 hook 系统，普通 App 能不能拿到 Flyme 原生实况通知？** 答案：能。

## 实测结果

| 实验 | 结果 | 证据 |
|---|---|---|
| 普通通知 + live extras 出胶囊 | ✅ | 状态栏出现胶囊（图标+文字+自适应底色） |
| 白名单写入即准入（无签名校验） | ✅ | 第三方包名写入后系统接受 |
| 胶囊背景跟随 App 图标颜色 | ✅ | 取图标平均色 → 浅蓝/粉色自适应，与源 App 一致 |
| 胶囊图标 = App 图标 | ✅ | 酷狗 K / bilibili 图标正常显示 |
| 实况卡片显示专辑封面 | ✅ | iconOverlay 放封面 Bitmap 96×96 |
| 卡片按钮控制播放 | ✅ | extendImage ↔ actions 映射，播放/暂停/上下曲生效 |
| 委托包名（关闭源 App 动画归属） | ✅ | `notification.live.app.pkg.delegated` |
| 多路并存（B站+酷狗+概念版） | ✅ | 3 条实况通知同时存在，各自独立卡片与按钮 |
| 关掉 App 后通知自动消失 | ✅ | 会话 STOPPED/NONE → 清槽位 + cancel |
| MediaStyle 与胶囊并存 | ❌ | MediaStyle 会让胶囊消失（二选一） |
| 视频类 App（B站）自动镜像 | ✅ | 会话名 `HeadsetControlPlayAdapter.Session`，状态 PLAYING、元数据=视频标题 |
| 非媒体 App（美团 ETA） | ❌ | 无 MediaSession，只能逐家解析通知文本 |

## 关键结论

1. **应用层可行**：SystemUI 只认通知字段，不校验 App 身份（签名校验未发现）
2. **媒体类 App 零适配**：`MediaSessionManager.getActiveSessions()` 自动发现，元数据/封面/状态都能读
3. **前台优先可做**：`UsageStatsManager` 查最近 RESUMED 的包名 → 优先镜像正在看的 App
   （需要 `PACKAGE_USAGE_STATS`，root 可 `appops set ... GET_USAGE_STATS allow` 授予）
4. **B站这类视频 App 也能抓**：它的会话是 `HeadsetControlPlayAdapter.Session`，别按包名写死过滤
5. **失败场景**：非媒体 App 没有会话；跨系统厂商完全不通用
