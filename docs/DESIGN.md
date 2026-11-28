# UI 视觉与尺寸规范

项目用 `com.zhy:autolayout:1.4.5` 做屏幕适配，设计稿基准 **720 × 1280**。布局里的 `px` 会被框架按屏幕宽度等比换算，所以写 `px` 而不是 `dp`。

## 1. 布局尺寸

| 元素 | 值 | 说明 |
|---|---|---|
| 设计稿宽度 | 720px | `AndroidManifest` 里 `design_width = 720` |
| 设计稿高度 | 1280px | `design_height = 1280` |
| 页面左右边距 | 32px | `activity_horizontal_margin` |
| 列表项左右边距 | 25px | `item_margin_left_and_right` |
| 列表项上下边距 | 15px | `item_margin_top_and_bottom` |
| 底部 Tab 图标 | 58 × 54px | `main_bottom_iv_horizontal_size` / `_vertical_size` |
| 头像（会话列表） | 90 × 90px | 圆角由 `CircleImageView` 裁切 |

写布局时优先引用 `dimens.xml` 里已有的值，新增尺寸也加进 `dimens.xml`，不要散落在各个布局里。

## 2. 字号

实际使用分布（自 `res/layout` 统计）：

| 字号 | 出现次数 | 场景 |
|---|---|---|
| `12sp` | 67 | 次要信息、时间戳、备注 |
| `14sp` | 34 | 列表主标题、按钮 |
| `16sp` | 29 | 页面标题、输入框 |
| `24px` | 23 | 工具栏标题（随屏幕缩放） |
| `28px` | 22 | 大标题、空态文案 |
| `15sp` | 10 | 消息气泡内文字 |

规则：**正文与标题用 `sp`**（跟随系统字体大小），**工具栏与图标旁的尺寸用 `px`**（跟随屏幕缩放）。气泡文字必须用 `sp`，否则用户调大字体后会溢出。

## 3. 颜色

主题色定义在 `res/values/colors.xml`：

| 名称 | 值 | 用途 |
|---|---|---|
| `main` | `#26AAFD` | 主色，按钮、链接、选中态 |
| `colorPrimary` | `#393A3F` | 工具栏背景（深灰，接近微信） |
| `colorPrimaryDark` | `#303135` | 状态栏 |
| `main_bg0` | `#F0F0F0` | 页面背景 |
| `main_bg1` | `#F8F8F8` | 分组背景 |
| `main_bg4` | `#F6F5F5` | 输入栏背景 |
| `line` | `#D2D2D2` | 分割线 |
| `assist_red` | `#FF6D6D` | 未读角标、错误提示 |
| `assist_green` | `#6AE068` | 发送成功 |
| `gray0` / `gray1` / `gray2` | `#464646` / `#7E7E7E` / `#969696` | 正文 / 次要 / 禁用 |

**不要硬编码颜色**，包括 `#fff`、`#000`，统一引用 `@color/xxx`。红包相关的红色用 `assist_red`，不要另起一套。

## 4. 主题

`res/values/styles.xml`：

| 样式 | 父主题 | 用途 |
|---|---|---|
| `AppTheme` | `Theme.AppCompat.Light.NoActionBar` | 默认主题，含 Activity 切换动画 |
| `FullWindowTheme` | 同上 + `windowFullscreen` | 启动页、图片查看 |
| `MainTheme` | 同上，无 colorPrimary | 主界面四个 Tab |

页面标题栏不是原生 `Toolbar`，而是 `BaseActivity` 里用 `FrameLayout + ImageView + TextView` 拼的，所以**改标题栏要改 `activity_base.xml` 及所有引用它的布局**，不要去动 `Toolbar` 控件。

## 5. 交互与动效

- Activity 切换动画统一 `@style/activityAnimation`（左右平移），不要单个页面自定义
- 长按菜单用 `CustomDialog`，不要用 `PopupMenu`，样式与微信一致
- 加载中统一 `showWaitingDialog()`，禁止用 `ProgressBar` 铺满页面
- 空列表统一提示文案在 `strings.xml`，不要写"暂无数据"四个字到布局里

## 6. 自定义控件

| 控件 | 位置 | 说明 |
|---|---|---|
| `LetterIndexBar` | `widget` | 联系人右侧字母索引，按下时中间浮出大字母 |
| `CustomDialog` | `widget` | 通用对话框，长按菜单与确认框都用它 |
| `ChatLinearLayout` | `widget` | 消息气泡容器，处理左右对齐与最大宽度 |
| `PortraitImageView` | `widget` | 圆形头像，带默认占位图 |

新增控件放 `widget` 包，布局里用全限定名引用，不要塞进 `ui/view`（那里是 MVP 的 View 接口）。

## 7. 检查清单

- [ ] 布局里没有硬编码颜色与字号
- [ ] 新增尺寸进了 `dimens.xml`
- [ ] 文案进了 `strings.xml`，英文版（`values-en`）可选
- [ ] 720×1280 下不溢出，1080×1920 下不变形
- [ ] 气泡文字用 `sp`，长文本能换行不截断
