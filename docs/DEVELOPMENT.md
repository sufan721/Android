# 开发环境说明

本项目基于早期 Android 工具链，环境版本比较敏感，建议按下面表格配置。

## 1. 环境要求

| 项 | 版本 | 说明 |
|---|---|---|
| JDK | **8** | AGP 2.3 与 retrolambda 均不支持 JDK 11+，用 11 及以上会直接编译失败 |
| Gradle | 3.3 | 已在 `gradle/wrapper` 中固定，无需本地安装 |
| Android Gradle Plugin | 2.3.0 | 见根目录 `build.gradle` |
| compileSdkVersion | 25 | 对应 Android SDK Platform 25 |
| buildToolsVersion | 25.0.2 | 需在 SDK Manager 中单独安装 |
| minSdkVersion | 16 | 覆盖到 Android 4.1 |

> 如果使用的是较新的 Android Studio，请在 `Settings → Build, Execution, Deployment → Build Tools → Gradle` 中把 Gradle JDK 指定为 JDK 8。

## 2. 拉取代码

项目的 7 个 UI 依赖库以 git submodule 形式放在 `libraries/` 下，必须连同子模块一起拉取：

```bash
git clone --recursive https://github.com/sufan721/Android.git

# 已经 clone 过但目录为空时
git submodule update --init --recursive
```

拉取完成后 `libraries/LQRAdapterLibrary`、`libraries/LQRImagePicker` 等目录应当有源码，否则 `settings.gradle` 会报找不到 project。

## 3. 必须替换的第三方配置

项目依赖融云、腾讯地图、ShareSDK 等第三方服务，仓库里只保留了占位的调试值，运行前需要换成自己的：

1. 融云 AppKey：`app/src/main/AndroidManifest.xml` 中的 `RONG_CLOUD_APP_KEY`
2. 腾讯地图 Key：`app/src/main/AndroidManifest.xml` 中的 `TencentMapSDK`
3. ShareSDK 各平台 Key：`app/src/main/assets/ShareSDK.xml`
4. 服务端地址：`app/src/main/java/com/lqr/wechat/api/MyApi.java` 中的 `BASE_URL`，原始地址已不可用，需自行部署 [sealtalk-server](https://github.com/rongcloud/sealtalk-server)
5. 支付宝红包签名：`app/src/main/java/com/lqr/wechat/app/MyApp.java` 中的 `initRedPacket()`

## 4. 构建与安装

```bash
# 编译 Debug 包
./gradlew :app:assembleDebug

# 安装到已连接的设备
./gradlew :app:installDebug
# 或
adb install -r app/build/outputs/apk/app-debug.apk
```

产物位于 `app/build/outputs/apk/`。

## 5. 关于 so 文件

`app/build.gradle` 中通过 `abiFilters` 只打包 `armeabi-v7a` 与 `arm64-v8a`。融云 SDK 未提供 x86_64 的 so，如果强行全量打包，在 x86_64 模拟器上会在加载 native 库时崩溃。

编译过程中的常见报错见 [FAQ.md](FAQ.md)。
