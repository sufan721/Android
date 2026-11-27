# 常见问题

## 1. 编译报 `Unsupported class file major version` 或 retrolambda 相关错误

用的是 JDK 11 及以上。本项目锁定 AGP 2.3.0 + retrolambda 3.2.5，只能跑 JDK 8。请在 IDE 中把 Gradle JDK 切回 8。

## 2. 同步时报找不到 `:lqr-adapter` 等 project

`libraries/` 下的 7 个库是以 submodule 引入的，目录是空的：

```bash
git submodule update --init --recursive
```

## 3. 依赖拉不下来（AutoLayout、LitePal 1.5 等）

这些库原来托管在 jcenter，jcenter 已下线，仓库里的 `jcenter()` 只是占位。可行的做法：

- 换用镜像源（如阿里云 maven 镜像）后在 `build.gradle` 中替换仓库地址；
- 或把这些依赖升级到 `mavenCentral()` 上仍可用的版本，注意 API 变更。

## 4. 登录一直失败

`MyApi.BASE_URL` 指向的 `api.sealtalk.im` 已经停止服务。需要自行部署 [sealtalk-server](https://github.com/rongcloud/sealtalk-server)，然后把 `BASE_URL` 改成你的地址（注意结尾保留 `/`）。

## 5. 启动即崩溃，日志里是融云初始化失败

`AndroidManifest.xml` 中的 `RONG_CLOUD_APP_KEY` 还是占位值。到融云控制台创建应用，把 AppKey 换进去。

## 6. 地图相关页面空白 / 定位失败

`TencentMapSDK` 还是占位值，需要替换成腾讯位置服务申请的 Key，并在控制台为该 Key 配置对应的包名与签名 SHA1。

## 7. 分享面板点开闪退

`app/src/main/assets/ShareSDK.xml` 中各平台（微信、QQ、微博）的 AppId/AppSecret 未配置。至少把要用的平台填上，未使用的平台不要保留空节点。

## 8. 红包功能不可用

红包依赖支付宝的签名服务，`MyApp.initRedPacket()` 里需要配置自己的签名方式与服务端地址；另外该模块在项目里属于实验性质，未做完整联调。

## 9. x86 模拟器上装得上但打不开

融云 SDK 没有提供 x86_64 的 so，`app/build.gradle` 里只打包了 `armeabi-v7a` 与 `arm64-v8a`。请用 ARM 真机或 ARM 镜像的模拟器。

## 10. 打包出来体积很大

`app/libs` 中的 SDK jar 与 so 占了主要体积，且当前 `release` 未开启混淆（`minifyEnabled false`）。正式包可在 `app/build.gradle` 中开启 minify 并按需精简 ABI。
