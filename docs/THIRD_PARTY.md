# 三方 SDK 与依赖清单

本项目依赖大量闭源 SDK，其中一部分以 jar / aar 的形式直接放在 `app/libs`，另一部分通过 Gradle 坐标拉取。本文档记录它们的来源、版本与授权情况，便于审计与升级。

## 1. app/libs 中的本地二进制

这些文件不在任何 Maven 仓库里，升级只能手动替换。

| 文件 | 提供方 | 版本 | 用途 | 授权 |
|---|---|---|---|---|
| `Rong_IMLib.jar` | 融云 | IMKit/IMLib 2.x | 即时通讯：连接、会话、消息 | 商用需在融云后台申请，遵循融云服务协议 |
| `Rong_IMKit` 相关 so（`armeabi-v7a` / `arm64-v8a`） | 融云 | 同上 | 语音消息编解码 | 同上；**没有 x86_64 版本** |
| `alipaySdk-20161129.jar` | 支付宝 | 20161129 | 支付 | 支付宝开放平台协议 |
| `redpacket-alipay-1.1.2.aar` | 支付宝红包 | 1.1.2 | 红包消息与领取 | 同上 |
| `TencentMapSDK_Raster_v_1.2.7_51ae0e7.jar` | 腾讯位置服务 | 1.2.7 | 地图展示、位置共享 | 腾讯位置服务协议，key 绑定包名 |
| `TencentLocationSDK_v4.9.7.12_r247861_161205_1104.jar` | 腾讯位置服务 | 4.9.7.12 | 定位 | 同上 |
| `TencentSearch1.1.3.jar` | 腾讯位置服务 | 1.1.3 | POI 搜索 | 同上 |
| `ShareSDK-*-2.8.2.jar`（16 个） | Mob | 2.8.2 | 三方分享与登录（微信、QQ、微博、Facebook、Twitter 等） | Mob 开发者协议，各渠道另有平台授权 |
| `MobTools-2017.0322.0931.jar` | Mob | 2017.0322 | ShareSDK 基础库 | 同上 |
| `MobCommons-2017.0322.0931.jar` | Mob | 2017.0322 | ShareSDK 公共库 | 同上 |
| `pinyin4j-2.5.0.jar` | 开源 | 2.5.0 | 汉字转拼音，联系人首字母分组 | 见项目原始授权（GPL/自定义双授权，商用前请确认） |
| `joda-time-2.5.jar` | Joda | 2.5 | 时间计算 | Apache License 2.0 |
| `volley-1.0.19.jar` | Google | 1.0.19 | 部分遗留请求 | Apache License 2.0 |

**新增或替换这些文件时，必须同步更新本表**，并在 PR 描述里说明替换原因与版本差异。

## 2. Gradle 坐标依赖

| 依赖 | 版本 | 用途 |
|---|---|---|
| `com.android.support:appcompat-v7 / design / cardview-v7` | 25.3.1 | 基础 UI |
| `com.android.support:multidex` | 1.0.1 | 分包 |
| `com.squareup.retrofit2:retrofit / converter-gson / adapter-rxjava` | 2.2.0 | 网络层 |
| `com.squareup.okhttp3:okhttp / logging-interceptor` | 3.6.0 | HTTP 客户端 |
| `io.reactivex:rxandroid` | 1.2.1 | 线程调度 |
| `org.litepal.android:core` | 1.5.0 | 本地数据库 |
| `com.alibaba:fastjson` | 1.2.29 | JSON 解析（`JsonMananger`） |
| `com.qiniu:qiniu-android-sdk` | 7.3.3 | 图片与语音上传 |
| `com.jakewharton:butterknife` | 7.0.1 | 视图绑定（`@Bind`） |
| `com.github.bumptech.glide:glide` | 3.7.0 | 图片加载 |
| `com.google.zxing:core` | 3.2.1 | 二维码 |
| `cn.bingoogolapple:bga-qrcodecore / bga-zxing / bga-refreshlayout` | 1.1.7 | 扫码与下拉刷新 |
| `com.zhy:autolayout` | 1.4.5 | 720×1280 屏幕适配 |
| `com.jaeger.statusbaruitl:library` | 1.3.5 | 状态栏 |
| `me.drakeet.materialdialog:library` | 1.3.1 | 对话框 |
| `com.kyleduo.switchbutton:library` | 1.4.4 | 开关控件 |
| `de.hdodenhof:circleimageview` | 2.1.0 | 圆形头像 |
| `com.lovedise:permissiongen` | 0.0.6 | 运行时权限 |
| `com.bm.photoview:library` | 1.4.1 | 图片查看 |
| `cjt.library.wheel:camera` | 0.1.9 | 拍照 |

## 3. git submodule 引入的自研库

`settings.gradle` 通过 submodule 引入以下 7 个库（jcenter 下线后改为源码依赖）：

| 模块 | 仓库 | 用途 |
|---|---|---|
| `lqr-adapter` | LQRAdapterLibrary | RecyclerView 通用适配器 |
| `lqr-recyclerview` | LQRRecyclerViewLibrary | 列表封装 |
| `lqr-imagepicker` | LQRImagePicker | 相册选图 |
| `lqr-ninegridimageview` | LQRNineGridImageView | 九宫格图片 |
| `lqr-optionitemview` | LQROptionItemView | 设置项控件 |
| `lqr-emoji` | LQREmojiLibrary | 表情键盘 |
| `lqr-audio` | LQRAudioRecord | 录音 |

克隆后必须执行：

```bash
git submodule update --init --recursive
```

未初始化时 `settings.gradle` 会因找不到 `libraries/*/library` 目录而编译失败。

## 4. 授权合规提醒

1. 仓库公开不代表这些 SDK 可用于商业分发：融云、支付宝、腾讯位置服务、Mob 都需要以自己的账号申请 key 与服务。
2. `AndroidManifest.xml` 里的 `RONG_CLOUD_APP_KEY`、`TencentMapSDK` 是示例值，正式发布请替换；腾讯地图 key 与包名、签名绑定，换签名后需要重新申请。
3. 本项目 UI 与交互参考微信，仅用于学习与技术演示，请勿用于商业仿冒。
4. 升级任何一个 SDK 前，先确认其对新版 Android（尤其 Android 6.0 运行时权限、Android 7.0 FileProvider、Android 8.0 后台限制）的支持情况。
