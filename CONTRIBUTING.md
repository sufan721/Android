# 贡献指南

欢迎提 Issue 和 PR。为了让改动更容易被 review，麻烦遵守下面几点。

## 分支约定

| 前缀 | 用途 | 示例 |
|---|---|---|
| `feat/` | 新功能 | `feat/group-qrcode` |
| `fix/` | 缺陷修复 | `fix/audio-record-crash` |
| `docs/` | 文档 | `docs/build-guide` |
| `refactor/` | 重构，不改行为 | `refactor/session-adapter` |

请从 `main` 切分支，一个 PR 只做一件事。

## 提交信息

格式：`type(scope): subject`

- `type`：`feat` / `fix` / `docs` / `style` / `refactor` / `chore`
- `scope`：模块名，如 `session`、`contacts`、`net`、`ui`
- `subject`：中文或英文均可，动词开头，不加句号

```
feat(session): 支持长按消息复制
fix(net): 修复 Cookie 过期后不重新登录
docs: 补充 JDK 版本要求
```

## 代码风格

- Java 4 空格缩进，单个方法不超过 80 行
- Activity/Fragment 不直接发请求，请求写在对应的 `*Presenter` 中，回调通过 `I*View` 暴露
- 新增页面需要同时提供：Activity、Presenter、`I*View`、布局文件
- 新增控件放 `widget/`，通用逻辑抽到 `util/`，不要在 Activity 里堆工具方法

## PR 流程

1. Fork 后从 `main` 切分支
2. 改动尽量带上截图或录屏（界面相关）
3. 提交前自测：`./gradlew :app:assembleDebug` 能过
4. PR 描述里写清楚：改了什么、为什么改、影响范围

## 关于第三方 SDK

仓库内的 `app/libs` 是融云、支付宝、ShareSDK 等闭源 SDK，请勿替换或升级版本，以免引入签名与 so 兼容问题。
