# 消息类型与协议约定

聊天功能全部基于融云 SDK（`Rong_IMLib.jar` + `sdk/RongCloudSDK`），HTTP 接口只负责好友与群资料，不参与消息收发。

## 1. 连接与会话

```java
// MyApp#onCreate 里初始化
RongCloudSDK.init(context);
// 登录成功后连接
RongIMClient.connect(token, callback);
```

- 会话类型只用两种：`ConversationType.PRIVATE`（单聊）、`ConversationType.GROUP`（群聊）
- `targetId`：单聊是对方 `userId`，群聊是 `groupId`
- 连接 token 来自 `/user/login` 返回的 `token`，与 Cookie 里的登录态是两套东西

## 2. 支持的七种消息

| 类型 | 融云类 | 客户端位置 | 说明 |
|---|---|---|---|
| 文本 | `TextMessage` | `SessionActivity` | 支持 emoji 与表情贴纸（贴纸以文本编码发送） |
| 图片 | `ImageMessage` | 同上 | 先传七牛拿 URL，再按原图/缩略图分字段发送 |
| 语音 | `VoiceMessage` | 同上 | AMR 格式，`LQRAudioRecord` 录音，最长 60 秒 |
| 视频 | `ImageMessage` + 本地缩略图 | 同上 | 融云没有独立视频类型，靠约定字段区分 |
| 位置 | `LocationMessage` | 同上 | 经纬度 + POI 名，地图用腾讯地图 SDK 渲染 |
| 红包 | `RedPacketMessage`（自定义） | `model/message` | 支付宝红包，透传订单信息 |
| 删除联系人 | `DeleteContactMessage`（自定义） | `model/message` | 单方删除好友后给对方发一条系统消息 |

通知类消息：`ContactNotificationMessage` 用于好友申请与同意，`InformationNotificationMessage` 用于群内系统提示（建群、改名、踢人）。

## 3. 自定义消息

自定义消息需注册，否则收到会退化成"未知消息"：

```java
RongIMClient.registerMessageType(RedPacketMessage.class);
RongIMClient.registerMessageType(DeleteContactMessage.class);
```

约定：

- `MessageContent` 子类的 `encode()` / `decode()` 必须对称，字段增删要保持向后兼容
- 自定义消息带 `extra` 时，键名统一小写下划线，如 `red_packet_id`
- 未识别的自定义消息要降级成"当前版本不支持该消息，请升级"，不能直接崩

## 4. 消息状态

| 状态 | 表现 | 处理 |
|---|---|---|
| 发送中 | 气泡旁转圈 | — |
| 发送成功 | 转圈消失 | — |
| 发送失败 | 红色感叹号 | 点击重发，长按可删除 |
| 已撤回 | 气泡替换为"你撤回了一条消息" | 两分钟内可撤回 |

未读数由融云 `Conversation.getUnreadCount()` 提供，进入会话时 `clearMessagesUnreadStatus`，不要本地自己累加。

## 5. 容易踩的坑

1. **群成员昵称要自己拼**：融云消息只带 `senderUserId`，渲染群消息时要去 `GroupMember` 表查群内昵称，查不到再退回 `Friend` 的备注名。
2. **图片消息必须传缩略图**：只传原图时列表里会一直白块，`ImageMessage.obtain(thumbUri, localUri)` 两个参数都要给。
3. **语音时长以 `VoiceMessage.getDuration()` 为准**，不要用本地录音计时，两者差 1 秒很常见。
4. **红包消息不要复用文本通道**：早期实现用文本带 JSON，导致长按复制能拿到订单号，已改为自定义消息。
5. **会话列表刷新靠广播**：消息到达后 `BroadcastManager` 发 `UPDATE_RECENT_LIST`，`RecentMessageFg` 收到才刷新，不要在每个页面各自轮询。
