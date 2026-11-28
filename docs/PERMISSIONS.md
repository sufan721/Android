# 权限清单与申请时机

`AndroidManifest.xml` 里声明了 21 个权限，其中 6 个属于危险权限，需要在运行时申请（`minSdkVersion 16`，Android 6.0 以下用户安装即授予）。

运行时申请统一用 `com.lovedise:permissiongen` 的 `PermissionGen`，写在 `BaseActivity` 里，子类只实现回调。

## 1. 危险权限（需运行时申请）

| 权限 | 用在哪 | 申请时机 | 被拒绝后的表现 |
|---|---|---|---|
| `CAMERA` | 拍照发图、扫一扫 | 点击拍摄/扫码按钮时 | 提示"未获得相机权限"，不进入预览 |
| `RECORD_AUDIO` | 语音消息 | 首次按住说话前 | 提示后禁止录音按钮，长按无反应 |
| `WRITE_EXTERNAL_STORAGE` | 保存图片、选图、录音缓存 | 发送图片/保存到相册时 | 提示"需要存储权限"，图片选择器不可用 |
| `ACCESS_FINE_LOCATION` | 位置消息、实时位置 | 进入位置页时 | 地图可看，但定位不返回 |
| `ACCESS_COARSE_LOCATION` | 同上（网络定位兜底） | 同上 | 与上一条一起申请 |
| `READ_PHONE_STATE` | 融云 SDK 内部读取设备信息 | 登录页初始化时 | 不影响登录，SDK 日志会警告 |

申请写法：

```java
PermissionGen.with(this)
        .addRequestCode(REQUEST_CODE_PERMISSION)
        .permissions(Manifest.permission.CAMERA, Manifest.permission.WRITE_EXTERNAL_STORAGE)
        .request();
```

注意 `PermissionGen` 0.0.6 用的是 `@PermissionSuccess` / `@PermissionFail` 注解回调，不是 `onRequestPermissionsResult` 重写，别两套混用。

## 2. 普通权限（安装即授予）

| 权限 | 用途 |
|---|---|
| `INTERNET` | 网络请求、融云长连接 |
| `ACCESS_NETWORK_STATE` | 网络状态判断，断网时提示 |
| `ACCESS_WIFI_STATE` / `CHANGE_WIFI_STATE` | 融云链路优化、网络切换重连 |
| `CHANGE_NETWORK_STATE` | 断线重连 |
| `VIBRATE` | 新消息震动 |
| `WAKE_LOCK` | 融云后台保活、语音播放不休眠 |
| `MODIFY_AUDIO_SETTINGS` | 语音消息切换听筒/扬声器 |
| `RECEIVE_BOOT_COMPLETED` | 开机后恢复消息推送 |
| `GET_TASKS` | 判断应用是否在前台 |
| `GET_ACCOUNTS` / `MANAGE_ACCOUNTS` | ShareSDK 三方登录读取账号 |
| `BLUETOOTH` / `BLUETOOTH_ADMIN` | 语音消息蓝牙耳机切换 |
| `WRITE_SETTINGS` | 调整亮度（查看图片时） |

## 3. 申请时机原则

1. **用到才申请**，不在启动页一次性弹 6 个系统框
2. 申请前给用户一句话说明为什么要这个权限（自定义对话框），拒绝后不再反复弹
3. 拒绝且勾选"不再询问"时，引导去设置页：`Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS)`
4. 录音与相机权限被拒后，功能入口要置灰或提示，不能点了没反应

## 4. Android 版本适配

| 版本 | 影响 | 处理 |
|---|---|---|
| 6.0 | 危险权限运行时授予 | 上述 `PermissionGen` |
| 7.0 | `file://` URI 不能跨进程传递 | 拍照用 `FileProvider`，`authorities` 用 `${applicationId}.fileprovider` |
| 8.0 | 后台服务限制 | 融云推送走系统通道，不要自建前台服务 |
| 9.0 | 默认禁止明文 HTTP | 后端必须 HTTPS，否则登录直接失败 |
| 10 | 分区存储 | 保存图片改走 `MediaStore`，不再直接写 `Environment.getExternalStorageDirectory()` |

## 5. 新增权限的流程

- PR 描述里说明用途与申请时机，不能出现"为了省事先加上"
- 同步更新本文件的表格
- 能用系统选择器（如 `ACTION_PICK`）解决的，不要申请存储权限
- 不要申请 `READ_CONTACTS`、`READ_SMS` 这类与应用功能无关的高危权限，本项目刻意没有申请
