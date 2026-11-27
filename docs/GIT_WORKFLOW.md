# 分支模型与提交规范

## 1. 分支模型

仓库长期只有一条主分支 `main`，所有改动通过 Pull Request 合入。

| 分支 | 用途 | 生命周期 |
|---|---|---|
| `main` | 可运行的主线，每次合入都应能编译通过 | 长期 |
| `feat/<scope>` | 新功能 | 合并后删除 |
| `fix/<scope>` | 缺陷修复 | 合并后删除 |
| `docs/<scope>` | 文档、规范 | 合并后删除 |
| `ci/<scope>` | 构建、CI、依赖维护 | 合并后删除 |
| `chore/<scope>` | 工程配置、清理 | 合并后删除 |

`<scope>` 用小写短横线，尽量对应模块名：

```
feat/session-message
fix/unread-count
docs/code-style
ci/build-workflow
```

禁止直接向 `main` 推送代码提交（本节即约定），需要临时绕过时要在 PR 描述里说明原因。

## 2. 一次 PR 只做一件事

- 一个 PR 解决一个模块或一个问题，标题能一句话说清
- 单个 PR 建议不超过 400 行 diff，超出请拆分
- 重构与功能改动不要混在同一个 PR：先合重构，再合功能
- 格式化（空格、换行、import）单独成 PR，避免淹没真实改动

## 3. 提交信息格式

```
<type>(<scope>): <subject>
```

- `type`：feat / fix / docs / style / refactor / perf / test / build / ci / chore
- `scope`：可选，模块名，如 `session`、`contacts`、`net`、`util`
- `subject`：中文描述，动词开头，不超过 30 字，结尾不加句号

示例：

```
feat(session): 支持长按删除会话
fix(contacts): 修复字母索引栏点击错位
docs: 添加开发环境与构建说明
ci: 添加 Android 构建工作流
```

正文写"为什么改"与"影响面"，不要罗列改了哪些文件（`git show` 里已有）：

```
fix(contacts): 修复字母索引栏点击错位

索引项高度未跟随 autolayout 缩放，720×1280 设计稿下
点击位置与字母偏差约 6px。改为按实测高度计算分区偏移。

影响范围：ContactsFg 的索引栏与列表联动。
```

## 4. 开发流程

```bash
# 1. 同步主线并开分支
git checkout main
git pull
git checkout -b feat/session-message

# 2. 开发，小步提交
git add -A
git commit -m "feat(session): 支持撤回两分钟内的消息"

# 3. 推送并开 PR
git push -u origin feat/session-message
gh pr create --base main --fill

# 4. Review 通过后合并（保留提交，或按需 squash）
gh pr merge --merge --delete-branch

# 5. 回到最新主线
git checkout main
git pull
```

## 5. PR 要求

- 标题遵循提交信息格式，Reviewer 从标题就能看出改了什么
- 描述里填写：改动目的、影响范围、自测方式、截图（涉及 UI 时）
- 关联 Issue：`Closes #12`
- 提交前自查：`./gradlew test` 与 `./gradlew assembleDebug` 通过
- 涉及 UI 的改动附 720×1280 与真机截图各一张
- 不提交 `local.properties`、签名文件、`build/` 产物（已在 `.gitignore` 覆盖）

## 6. 合并方式

| 场景 | 方式 | 结果 |
|---|---|---|
| 单个功能分支、提交清晰 | Merge commit | 保留每条提交与合并记录 |
| 提交过于零碎（如连续 `fix typo`） | Squash | 压成一条，写清最终改动 |
| 分支基于最新 `main` 且线性 | Rebase | 不产生合并提交 |

合并后立即删除远端分支，本地分支同步清理：

```bash
git branch -d feat/session-message
git fetch --prune
```

## 7. 版本与标签

- 版本号与 `app/build.gradle` 的 `versionName` 保持一致，`versionCode` 每次发版递增
- 发布合入后打标签：`git tag -a v1.0.0 -m "首个正式版本" && git push origin v1.0.0`
- 标签只在 `main` 上打，历史标签不删除、不移动
