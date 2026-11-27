# 项目结构

项目是单模块工程（`:app`）+ 7 个本地 submodule 库，业务代码集中在 `com.lqr.wechat` 下，采用 MVP 分层。

## 1. 目录总览

```
app/src/main/java/com/lqr/wechat/
├── api/            网络请求层
│   ├── MyApi.java          所有接口声明 + BASE_URL
│   ├── ApiRetrofit.java    请求单例
│   └── base/               Retrofit 封装、Cookie 持久化
├── app/            Application 入口与全局常量
├── db/             LitePal 本地库与实体（Friend / Groups / GroupMember）
├── manager/        广播管理、JSON 解析
├── model/          数据模型
│   ├── request/    请求体
│   ├── response/   响应体
│   ├── message/    自定义消息（红包、删除联系人）
│   └── data/  cache/  exception/  redpacket/
├── sdk/            融云 IM SDK 封装
├── thread/         线程池
├── ui/             界面层（MVP）
│   ├── activity/   26 个 Activity
│   ├── fragment/   会话、联系人、发现、我
│   ├── presenter/  与页面一一对应的 Presenter
│   ├── view/       I*View 接口，定义页面能力
│   ├── base/       BaseActivity / BaseFragment / BasePresenter
│   └── adapter/    列表适配器
├── util/           20 个工具类（图片、文件、时间、拼音、SP 等）
└── widget/         自定义控件（气泡图、对话框、字母索引栏等）
```

资源侧：`res/layout` 按 `activity_* / fragment_* / item_* / dialog_* / include_*` 命名，与页面和消息类型一一对应。

## 2. MVP 分层

```
Activity / Fragment  ──实现──▶  I*View（ui/view）
        │                              ▲
        │持有                          │回调
        ▼                              │
   *AtPresenter / *FgPresenter ────────┘
        │
        ▼
   MyApi（Retrofit） / RongCloudSDK / DBManager
```

- **View 接口**：`ui/view` 下每个 `I*View` 只声明本页面需要的能力，例如登录页的 `ILoginAtView` 提供 `loginSuccess()`、`loginError()`。
- **Presenter**：负责请求、数据转换与页面状态回调，命令式方法名统一为 `login()`、`loadConversation()` 这类动词。
- **基类**：`BaseActivity` / `BaseFragment` 统一处理布局注入、Presenter 创建与销毁、软键盘与沉浸式状态栏。

## 3. 数据来源

| 数据 | 通道 |
|---|---|
| 账号、好友、群组、群成员 | 自建 sealtalk-server（Retrofit 调用，`MyApi`） |
| 会话与消息 | 融云 IM SDK（`sdk/RongCloudSDK` 封装），不落自建服务 |
| 本地缓存 | LitePal（好友、群组、群成员）与 SharedPreferences（登录态、当前用户） |

网络层的 Cookie 由 `api/base/persistentcookiejar` 持久化到 SharedPreferences，保证重启后会话不丢。

## 4. 消息类型与布局

| 类型 | 布局 |
|---|---|
| 文本 | `item_text_send.xml` / `item_text_receive.xml` |
| 图片 | `item_image_send.xml` / `item_image_receive.xml` |
| 语音 | `item_audio_send.xml` / `item_audio_receive.xml` |
| 视频 | `item_video_send.xml` / `item_video_receive.xml` |
| 位置 | `item_location_send.xml` / `item_location_receive.xml` |
| 表情贴纸 | `item_sticker_send.xml` / `item_sticker_receive.xml` |
| 红包 | `item_red_packet_send.xml` / `item_red_packet_receive.xml` |
| 通知 | `item_notification.xml` |
| 未支持 | `item_no_support_msg_type.xml` |

新增消息类型时，需要同时补充：布局文件、`SessionAtPresenter` 中的类型分发、以及 `RongCloudSDK` 里的消息注册。
