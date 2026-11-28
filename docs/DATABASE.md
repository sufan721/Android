# 本地数据库说明

本地只有三张表，用 LitePal 1.5.0 管理，数据库文件 `lqr_wechat.db`，版本 `1`。

配置在 `app/src/main/assets/litepal.xml`，新增实体必须在这里登记，否则表不会被创建：

```xml
<litepal>
    <dbname value="lqr_wechat"></dbname>
    <version value="1"></version>
    <list>
        <mapping class="com.lqr.wechat.db.model.Friend"></mapping>
        <mapping class="com.lqr.wechat.db.model.Groups"></mapping>
        <mapping class="com.lqr.wechat.db.model.GroupMember"></mapping>
    </list>
</litepal>
```

初始化在 `MyApp#onCreate`：`LitePal.initialize(this);`。

## 1. Friend（好友）

`com.lqr.wechat.db.model.Friend`，继承 `DataSupport`，实现了 `Comparable<Friend>` 用于拼音排序。

| 字段 | 类型 | 说明 |
|---|---|---|
| `userId` | String | 用户 id，融云 userId 与服务端 id 一致 |
| `name` | String | 原始昵称 |
| `portraitUri` | String | 头像地址 |
| `displayName` | String | 备注名，为空时列表显示 `name` |
| `region` | String | 手机号区号 |
| `phoneNumber` | String | 手机号 |
| `status` | String | 好友状态 |
| `timestamp` | Long | 关系更新时间，增量同步用 |
| `letters` | String | 首字母，索引栏分组依据，如 `L` |
| `nameSpelling` | String | 昵称全拼，搜索用 |
| `displayNameSpelling` | String | 备注名全拼，搜索用 |

排序规则：先比 `letters`，相同再比拼音全拼，`#` 组排最后。

```java
DataSupport.findAll(Friend.class);          // 取全部，再 Collections.sort()
DataSupport.where("userId = ?", userId).find(Friend.class);
```

## 2. Groups（群）

`com.lqr.wechat.db.model.Groups`，实现 `Serializable`，用于页面间传递。

| 字段 | 类型 | 说明 |
|---|---|---|
| `groupId` | String | 群 id |
| `name` | String | 群名 |
| `portraitUri` | String | 群头像 |
| `displayName` | String | 我在群里的昵称 |
| `role` | String | `0` 群主、`1` 群成员（源码注释如此，注意与常见约定相反） |
| `bulletin` | String | 群公告 |
| `timestamp` | String | 更新时间 |
| `nameSpelling` | String | 群名全拼，排序用 |

`role` 的判断散落在群设置页，改动时要全局搜索 `role`，不要只改一处。

## 3. GroupMember（群成员）

`com.lqr.wechat.db.model.GroupMember`，实现 `Serializable`。**群 id 冗余在每条成员记录上**，没有单独的关联表。

| 字段 | 类型 | 说明 |
|---|---|---|
| `userId` | String | 成员 id |
| `name` | String | 成员昵称 |
| `portraitUri` | String | 头像 |
| `groupId` | String | 所属群 id，查询时作为条件 |
| `displayName` | String | 群内昵称 |
| `nameSpelling` | String | 昵称全拼 |
| `displayNameSpelling` | String | 群内昵称全拼 |
| `groupName` | String | 群名冗余，避免联表 |
| `groupNameSpelling` | String | 群名全拼 |
| `groupPortrait` | String | 群头像冗余 |

按群查成员：

```java
DataSupport.where("groupId = ?", groupId).find(GroupMember.class);
```

## 4. 约定与坑

1. **聊天记录不入本地库**：消息由融云 SDK 自己持久化，这里只缓存好友与群资料。所以删除应用数据后聊天记录还在，但通讯录要重新同步。
2. **字段冗余是故意的**：`GroupMember` 里带群名与群头像，是为了成员列表能直接渲染，不用再查一次 `Groups`。改群名时要同步更新两张表，否则成员列表显示旧群名。
3. **拼音字段必须在写入时算好**：`letters`、`nameSpelling` 由 `PinyinUtils` 生成，不是 LitePal 自动维护。新增好友时如果漏了这两个字段，索引栏会把它丢进 `#` 组。
4. **升级要改版本号**：`litepal.xml` 的 `version` 不递增，改表结构后老用户不会触发迁移，直接抛 `SQLiteException`。
5. **不要在主线程做批量写入**：同步全量好友是循环 `saveOrUpdate`，放在 `thread` 包里的线程池执行。
