# 代码风格与命名规范

本文档约定本项目（`com.lqr.wechat`）的 Java 与资源写法。新增代码请尽量与既有代码保持一致，Review 时会按此表逐项检查。

## 1. 语言与基础约束

| 项 | 约定 |
|---|---|
| 语言级别 | Java 8（`sourceCompatibility 1.8`），通过 retrolambda 回填到低版本设备 |
| 最低系统 | `minSdkVersion 16`，不要使用 API 17+ 才有的方法而不做版本判断 |
| 目标系统 | `targetSdkVersion 25` |
| 依赖注入 | 不使用，对象由使用方直接 new 或通过单例获取 |
| 视图绑定 | ButterKnife 7.0.1，注解是 `@Bind`（不是 8.x 的 `@BindView`） |
| 多 Dex | 已开启 `multiDexEnabled`，分包不需要额外配置 |

API 级别判断统一走 `Build.VERSION.SDK_INT`，并且只在必要处判断：

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    // 5.0 才有的行为
}
```

## 2. 包结构与分层

```
com.lqr.wechat
├── api          Retrofit 接口与 Cookie 持久化（api/base）
├── app          Application、全局常量（app/base：MyApp、MyConstants）
├── db/model     LitePal 实体，继承 DataSupport
├── manager      BroadcastManager、JsonMananger 等跨页面协作
├── model        业务模型（request / response / message / cache / exception）
├── sdk          融云等三方 SDK 的封装入口
├── thread       线程池
├── ui
│   ├── activity      页面
│   ├── fragment      主界面四个 Tab 与可复用片段
│   ├── adapter       RecyclerView / BaseAdapter 适配器
│   ├── base          BaseActivity、BaseFragment、BasePresenter
│   ├── presenter     各页面的 Presenter
│   └── view          各页面的 View 接口
├── util         无状态工具类，类名以 Utils 结尾
└── widget       自定义控件
```

分层规则：**Activity/Fragment 只做视图操作与事件转发，数据读写放 Presenter，网络与数据库调用不直接写在 UI 层。**

## 3. 命名

### 3.1 类名

| 类型 | 规则 | 示例 |
|---|---|---|
| Activity | `功能 + Activity` | `SessionActivity`、`CreateGroupActivity` |
| Fragment | `功能 + Fg` | `ContactsFg`、`RecentMessageFg` |
| Presenter | `页面 + AtPresenter` | `SessionAtPresenter`、`LoginAtPresenter` |
| View 接口 | `I + 页面 + AtView` | `ISessionAtView`、`ILoginAtView` |
| 适配器 | `数据 + Adapter` | `RecentMessageAdapter` |
| 数据实体 | 名词，不加后缀 | `Conversation`、`GroupMember` |
| 工具类 | `名词 + Utils` | `StringUtils`、`TimeUtils` |
| 自定义控件 | 名词 | `LetterIndexBar`、`CustomDialog` |

`AtPresenter` / `AtView` 中的 `At` 来自 "Activity"，是项目历史命名，新增页面请沿用，不要混入 `Presenter` / `View` 后缀。

### 3.2 变量与常量

- 成员变量加 `m` 前缀：`mPresenter`、`mData`、`mAdapter`
- 静态变量加 `s` 前缀：`sContext`、`sHandler`
- 常量全大写，下划线分词：`SP_KEY_TOKEN`、`REQUEST_CODE_PICK_PHOTO`
- 局部变量不加前缀，用完整单词，不要 `a`、`b`、`tmp1`
- 集合命名用复数：`mConversationList`、`mSelectedMembers`

### 3.3 方法与回调

- 方法动词开头：`loadConversation()`、`refreshUnreadCount()`、`showWaitingDialog()`
- 布尔返回用 `is` / `has` / `can`：`isLogin()`、`hasNewFriend()`
- 接口回调方法用 `on` + 事件：`onLoadSuccess()`、`onSendMessageFailed()`
- 不要写 `doSomething()`、`handleData()` 这类含义空洞的名字

## 4. 代码格式

- 缩进 4 空格，不使用 Tab（见 `.editorconfig`）
- 左大括号不换行；`else` 与前后大括号同行
- 单行不超过 120 字符，超长链式调用按 `.` 换行并缩进 8 空格
- `if` / `for` / `while` 即使只有一行也必须带大括号
- import 不使用 `*`，按 `android` → 三方 → `com.lqr.wechat` 分组，组间空一行
- 未使用的 import、字段、方法在提交前删掉

## 5. 资源命名

| 类型 | 前缀 | 示例 |
|---|---|---|
| 布局 | `activity_` / `fragment_` / `item_` | `activity_session.xml`、`item_recent_message.xml` |
| 图标 | `ic_` | `ic_add.png`、`ic_photo_blue.png` |
| 选择器 | 控件 + 状态 | `btn_send_bg.xml` |
| 动画 | `anim_` | `anim_loading.xml` |
| id | 控件缩写 + 语义 | `tvTitle`、`ivAvatar`、`rvConversation` |

控件缩写统一：`tv` TextView、`iv` ImageView、`rv` RecyclerView、`lv` ListView、`et` EditText、`btn` Button、`ll` LinearLayout、`rl` RelativeLayout、`fl` FrameLayout。

文案**不要硬编码在布局里**，一律写进 `res/values/strings.xml`（218 条），需要英文的同步写 `values-en`。三方 SDK 自带的 `rc_*.xml`、`ssdk_*.xml` 不要改。

## 6. 注释

- 公开方法用 `/** */` 说明用途、参数、返回值
- 复杂逻辑（消息合并、未读数计算、拼音排序）必须写清"为什么这样做"
- 临时方案用 `// TODO:` 标注，并说明触发条件
- 不写"给变量赋值"这类废话注释

## 7. 提交前自检

1. 没有新增 `TODO` 却没有说明
2. 没有把 `Log.d` 的调试输出留在主流程里（保留必要的错误日志）
3. 布局在 720×1280 设计稿下不溢出（项目使用 `autolayout`，字号与间距写 `px` 由框架换算）
4. 新增文案已进 `strings.xml`
5. `./gradlew test` 与 `./gradlew assembleDebug` 能通过
