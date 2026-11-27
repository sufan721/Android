# 打包与发布说明

## 1. 版本号

版本信息集中在 `app/build.gradle` 的 `defaultConfig`：

```gradle
versionCode 1
versionName "1.0"
```

- `versionCode` 每次对外发版必须 +1，只能递增
- `versionName` 与 `CHANGELOG.md`、git tag 保持一致（如 `v1.0.0`）
- 调试包与正式包共用同一套版本号，靠 `applicationId` 之外的后缀区分时请自行添加 `applicationIdSuffix`

## 2. 构建前检查

清单里的三方 key 必须是**自己申请的**，不要沿用仓库里的示例值：

| 位置 | 名称 | 用途 | 申请地址 |
|---|---|---|---|
| `AndroidManifest.xml` | `RONG_CLOUD_APP_KEY` | 融云 IM，登录与消息收发全靠它 | 融云开发者后台 |
| `AndroidManifest.xml` | `TencentMapSDK` | 腾讯地图，位置与实时位置共享 | 腾讯位置服务控制台 |
| `app/src/main/java/.../api/MyApi.java` | `BASE_URL` | 后端接口根地址 | 自建服务 |
| 七牛相关配置 | AccessKey / SecretKey | 图片、语音上传 | 七牛云控制台 |
| ShareSDK | appKey / appSecret | 三方分享登录 | Mob 开发者后台 |

`RONG_CLOUD_APP_KEY` 不对时应用能启动，但登录会一直提示连接失败；`TencentMapSDK` 不对时地图白屏。这两个是发布前最容易漏的。

## 3. 构建命令

```bash
# 调试包
./gradlew clean assembleDebug
# 产物：app/build/outputs/apk/debug/app-debug.apk

# 正式包
./gradlew clean assembleRelease
# 产物：app/build/outputs/apk/release/app-release.apk（未配置签名时为 unsigned）
```

首次构建需要 `local.properties` 指定 SDK 路径：

```properties
sdk.dir=D:\\Android\\sdk
```

`local.properties` 已被 `.gitignore` 忽略，不要提交。

## 4. 签名

当前工程**没有配置签名**：`buildTypes.release` 只引用了 `proguard-rules.pro`，`minifyEnabled false`，因此 `assembleRelease` 产出的是未签名包，无法直接安装到真机以外分发。

本地签名建议放在 `signing.properties`（已在 `.gitignore` 中忽略），由 `app/build.gradle` 读取：

```properties
storeFile=../keystore/release.jks
storePassword=******
keyAlias=wechat
keyPassword=******
```

```gradle
def signingProps = new Properties()
def signingFile = rootProject.file('signing.properties')
if (signingFile.exists()) {
    signingProps.load(new FileInputStream(signingFile))
}

android {
    signingConfigs {
        release {
            storeFile file(signingProps['storeFile'])
            storePassword signingProps['storePassword']
            keyAlias signingProps['keyAlias']
            keyPassword signingProps['keyPassword']
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

签名文件、密码、`*.jks`、`*.keystore`、`signing.properties` 一律不入库。仓库里已有 `*.jks` / `*.keystore` / `signing.properties` 的忽略规则。

## 5. 混淆

`minifyEnabled false`，`proguard-rules.pro` 目前是空模板。开启混淆前必须补规则，否则以下部分会崩：

- 融云 SDK：`Rong_IMLib.jar` 的 `io.rong.**` 及其 `Parcelable` 实现
- LitePal：`org.litepal.**`，实体类需 `-keep class * extends org.litepal.crud.DataSupport`
- Gson / Fastjson：`model.request.**`、`model.response.**` 的字段名不能混淆
- ButterKnife 7.0.1：`@Bind` 依赖字段名反射，需 `-keep class **$$ViewBinder { *; }`
- 支付宝、ShareSDK、腾讯地图：按官方文档各补一段 `-dontwarn` / `-keep`

## 6. 打包体积与 ABI

`defaultConfig.ndk.abiFilters` 只保留 `armeabi-v7a` 与 `arm64-v8a`：

```gradle
ndk { abiFilters 'armeabi-v7a', 'arm64-v8a' }
```

原因是融云 SDK 的 so 没有 x86_64 版本，强行打进包里会在 x86_64 设备上加载失败。同时已开启 `multiDexEnabled`，方法数超限不需要额外分包配置。

## 7. 发布流程

1. 从 `main` 拉发布分支，把 `versionCode` +1、`versionName` 改成目标版本
2. 更新 `CHANGELOG.md`，写清新增、修复、已知问题
3. 打 tag 并推送：`git tag -a v1.0.0 -m "v1.0.0" && git push origin v1.0.0`
4. 构建正式包，签名后校验：

```bash
# 校验签名
apksigner verify --verbose app-release.apk
# 查看包名与版本
aapt dump badging app-release.apk | head -3
```

5. 上传分发渠道，回填 `CHANGELOG.md` 的发布日期

## 8. 回滚

- 线上问题用已打过 tag 的历史提交重新构建，不要在旧 tag 上直接改代码
- `versionCode` 不允许回退，回滚版本要 +1 后重新发布
