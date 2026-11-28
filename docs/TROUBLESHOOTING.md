# 故障排查手册

按"现象 → 先查什么 → 常见原因"组织。所有排查都以 `adb logcat` 为第一现场。

```bash
# 只看本应用日志
adb logcat -s WeChat:V Rong:V AndroidRuntime:E
# 抓崩溃
adb logcat -b crash -d
# 清日志后复现
adb logcat -c && adb logcat > log.txt
```

## 1. 登录一直转圈或提示连接失败

**先查**：`AndroidManifest.xml` 里的 `RONG_CLOUD_APP_KEY` 是否为自己申请的 key。

| 原因 | 表现 | 处理 |
|---|---|---|
| 融云 appKey 是示例值 | 能输账号密码，点登录后一直转圈，logcat 里 `connect` 回调 31004 | 换成自己后台的 key |
| 融云后台未配置该包名/签名 | 连接直接失败，错误码 31003 | 后台登记 `applicationId` 与签名 MD5 |
| `BASE_URL` 不通 | 登录接口超时，`SocketTimeoutException` | 用浏览器直接访问 `BASE_URL` 验证 |
| 服务端 `code` 非 200 | 弹出"登录失败:xxx" | 按返回码查服务端，客户端只转述 |
| Android 9+ 明文 HTTP | 直接抛 `Cleartext HTTP traffic not permitted` | 后端上 HTTPS |

## 2. 地图白屏 / 定位不返回

- `TencentMapSDK` 的 key 与**包名 + 签名**绑定，换签名后必须重新申请，否则地图只出灰底
- 定位无返回先看是否给了 `ACCESS_FINE_LOCATION`，Android 10 之后还要 `ACCESS_BACKGROUND_LOCATION`（本项目未申请，仅前台定位可用）
- 模拟器无 GPS，要用真机或手动 mock 位置

## 3. 语音消息没声音 / 录音失败

| 现象 | 原因 |
|---|---|
| 长按无反应 | `RECORD_AUDIO` 被拒，`PermissionGen` 回调没处理 |
| 录出来的文件 0 字节 | 录音前没创建缓存目录，`WRITE_EXTERNAL_STORAGE` 被拒 |
| 播放走听筒声音很小 | `MODIFY_AUDIO_SETTINGS` 缺失或未切换 `AudioManager.MODE_NORMAL` |
| 蓝牙耳机无声 | `BLUETOOTH` 权限与 `startBluetoothSco` 时序问题，先断开重连 |

## 4. 图片消息一直白块

`ImageMessage.obtain(thumbUri, localUri)` 两个参数都要给：只传原图时，会话列表拿不到缩略图就渲染空白。另外七牛上传失败时本地 `localUri` 仍可用，不要把失败当成空路径丢掉。

## 5. 编译失败

| 报错 | 原因 | 处理 |
|---|---|---|
| `Could not resolve com.zhy:autolayout:1.4.5` | jcenter 已下线 | 换 `mavenCentral` / 阿里云镜像，或改用本地 aar |
| `Project with path ':lqr-adapter' could not be found` | submodule 未初始化 | `git submodule update --init --recursive` |
| `Unsupported class file major version` | JDK 版本过高 | 用 JDK 8 编译（`compileOptions` 是 1.8） |
| `Could not find method compile()` | Gradle 7+ 移除了 `compile` | 换回 Gradle 3.3，或把 `compile` 全改成 `implementation` |
| `Failed to find Build Tools revision 25.0.2` | SDK 未装 | `sdkmanager "build-tools;25.0.2"` |

## 6. 启动即崩

| 异常 | 原因 |
|---|---|
| `SQLiteException: no such table` | 改了实体字段但没升 `litepal.xml` 的 `version` |
| `ClassNotFoundException: ...MultiDex` | 5.0 以下设备未调用 `MultiDex.install()`，本项目已开启但自定义 Application 时容易漏 |
| `NullPointerException` in `MyApp#onCreate` | 融云 `init` 在 `LitePal.initialize` 之前，顺序不能换 |
| `FileUriExposedException` | Android 7 拍照用了 `file://`，改 `FileProvider` |

## 7. 会话列表不刷新

消息到达后靠 `BroadcastManager` 发 `UPDATE_RECENT_LIST` 广播。收不到通常是：

1. Fragment 已 `onDestroyView` 但没反注册广播（内存泄漏 + 收不到）
2. 广播 action 字符串拼错，搜 `AppConst` 里的常量统一引用
3. 在子线程改了集合却忘了 `notifyDataSetChanged` 回主线程

## 8. 二维码扫不出

- 扫码页未给 `CAMERA` 权限
- 扫码结果里的用户 id 与融云 `userId` 不一致（服务端换过 id 规则），表现为扫到了但查不到用户
- 生成二维码时带了过期时间，超过后解析为空

## 9. 开启混淆后崩溃

`minifyEnabled false` 时不会遇到；一旦开启必须补规则（见 [RELEASE.md](RELEASE.md) 第 5 节），最容易漏的是 LitePal 实体、Gson 字段名、ButterKnife 的 `$$ViewBinder`。

## 10. 提交前自查命令

```bash
./gradlew :app:assembleDebug --stacktrace   # 编译
./gradlew test                              # 单元测试
adb install -r app/build/outputs/apk/debug/app-debug.apk
adb shell am start -n com.lqr.wechat/.ui.activity.SplashActivity
```
