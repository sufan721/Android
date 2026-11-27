# 安全策略

## 1. 支持范围

本项目是学习性质的 Android 客户端源码（`com.lqr.wechat`），只对 `main` 分支的最新代码提供安全修复，不回补历史 tag。发现问题的版本请在报告中附上 `main` 的提交 SHA。

## 2. 报告漏洞

**请不要先开公开 Issue。** 任选一种方式：

1. 在本仓库的 Security 页面使用 "Report a vulnerability"（GitHub Security Advisory，推荐，支持私下沟通）
2. 发邮件到 `fsu67024@gmail.com`，标题前缀 `[SECURITY]`

报告里请包含：

- 受影响的页面或模块（如登录、红包、图片上传）
- 复现步骤，能复现的最小条件
- 影响评估：能读到什么数据、能否被远程触发
- 你使用的设备与系统版本

一般 3 个工作日内确认收到，修复后会在此文档与 `CHANGELOG.md` 中致谢（不愿署名的请说明）。

## 3. 已知的设计取舍

这是演示项目，以下问题**已知且不视为漏洞**，请勿重复提交：

- **服务端是自带的教学后端**，接口鉴权只有 token 校验，没有做防重放与频率限制
- **`BASE_URL`、`RONG_CLOUD_APP_KEY`、`TencentMapSDK` 等写在代码与清单里**：移动端无法真正隐藏密钥，正式发布请替换为自己的 key，并接受 key 可被反编译提取的事实
- **`minifyEnabled false`**：未开启混淆，反编译可直接读到类名与逻辑
- **HTTP 明文依赖**：部分三方 SDK 内部仍支持明文回退，是否强制 HTTPS 取决于后端配置
- **本地数据库 LitePal 未加密**：聊天记录以明文存在应用私有目录

## 4. 需要认真对待的问题

以下属于有效漏洞，请按第 2 节报告：

- 越权读取他人会话、群成员或好友关系
- 红包金额、支付回调可被客户端篡改
- 图片/语音上传路径可穿越到任意目录
- 二维码、深链接可被外部应用劫持并注入参数
- WebView（`WebViewActivity`）的 JS 接口暴露导致任意代码执行
- 签名校验缺失导致的可重打包分发

## 5. 提交代码时的安全要求

- 不提交真实签名文件、`signing.properties`、七牛 AccessKey/SecretKey、生产环境 `BASE_URL`
- WebView 只加载可信域名，关闭 `setAllowFileAccessFromFileURLs` 与 `setAllowUniversalAccessFromFileURLs`
- 涉及外部输入的地方（扫码结果、剪贴板、Intent extra）必须校验，不直接拼进 SQL 或 URL
- 新增权限时在 PR 里说明必要性，不要为了省事申请 `WRITE_EXTERNAL_STORAGE` 之类的宽泛权限
- 日志里不打印手机号、token、消息正文
