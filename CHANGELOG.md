# 更新日志

本项目遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

## [未发布]

本轮只补文档与工程配置，应用代码零改动。

### 新增

- **文档体系**：代码规范、Git 工作流、发布与签名、测试与自测清单、三方 SDK 清单、安全策略、接口说明、数据库表结构、消息协议、权限清单、UI 规范、故障排查、路线图、行为准则
- **工程配置**：`.editorconfig`、`.gitattributes`、完善的 `.gitignore`、Issue/PR 模板、Issue 入口配置
- **CI**：`android-build.yml`（JDK 8 + SDK 25，跑 `test` 与 `assembleDebug`）、`dependabot.yml`（Gradle 每周、Actions 每月）

### 变更

- 依赖经 Dependabot 升级：`retrofit 3.0.0`、`okhttp 5.5.1`、`qiniu-android-sdk 8.11.0`、`butterknife 10.2.3`、`gradle wrapper 9.9.2`
- GitHub Actions 升级：`checkout v7`、`setup-android v4`、`upload-artifact v7`

### 已知问题

- **升级后与 AGP 2.3.0 不兼容，仓库当前无法编译**：Gradle 9 不支持 AGP 2.x，`compile` 配置在 Gradle 7 已移除，butterknife 10 / qiniu 8 要求 AndroidX。回滚与迁移方案见 [路线图](docs/ROADMAP.md)
- CI 红：`setup-android` 的 sdkmanager 需要 JDK 17+；即使调整顺序，jcenter 下线也会让依赖解析失败
- `gradlew.bat` 存储为 CRLF 且与 `.gitattributes` 的 `eol=crlf` 规则冲突，clone 后可能显示为已修改

## [1.0.0] - 2026-11-26

首个完整版本，覆盖微信的核心社交功能。

### 新增

- **账号**：手机号注册、验证码校验、登录、修改密码
- **好友**：搜索用户、发送好友申请、处理新朋友、修改备注、加入黑名单、删除好友
- **通讯录**：好友列表、拼音索引、二维码名片、扫一扫添加
- **群组**：创建群聊、群列表、邀请成员、移除成员、修改群名称与群昵称、退群与解散
- **会话**：会话列表与未读角标、文本 / 图片 / 语音 / 视频 / 位置 / 表情贴纸 / 红包消息、消息长按菜单、会话设置
- **发现**：朋友圈入口、扫一扫
- **我**：个人资料编辑、头像与昵称修改、我的位置、设置、关于
- **工程**：MVP 分层基类、Retrofit + Cookie 持久化网络层、融云 IM 封装、LitePal 本地缓存

### 已知问题

- 依赖 jcenter 的若干库（AutoLayout、LitePal 1.5）已下线，需按 [FAQ](docs/FAQ.md) 替换源或升级版本后才能编译
- 红包模块仅完成流程联调，未做完整异常处理
- 朋友圈未实现

[1.0.0]: https://github.com/sufan721/Android/releases/tag/v1.0.0
