# 服务端接口说明

客户端所有请求都走 `com.lqr.wechat.api`，入口是单例 `ApiRetrofit`，接口声明在 `MyApi`。

```
Presenter ──> ApiRetrofit.login(region, phone, pwd) ──> MyApi.login(@Body RequestBody) ──> BASE_URL
                     ↑ 组装 XxxRequest 对象                  ↑ Retrofit + RxJava
```

- `BASE_URL` 定义在 `ApiRetrofit`，形如 `https://host/`，末尾必须带 `/`
- 请求体统一由 `XxxRequest` 模型 + `getRequestBody()` 序列化成 JSON
- 返回统一是 `Observable<XxxResponse>`，在 Presenter 里 `subscribeOn(io).observeOn(mainThread)`
- 登录态靠 Cookie，`api/base/persistentcookiejar` 做了持久化，进程重启后仍可用

## 1. 统一响应结构

```json
{ "code": 200, "result": { } }
```

| 字段 | 类型 | 说明 |
|---|---|---|
| `code` | int | `200` 表示成功，其余为失败，失败原因只体现在 code 上 |
| `result` | object / array | 业务数据，失败时可能为空 |

客户端只判断 `code == 200`，非 200 一律包成 `ServerException` 交给 View 层提示。**没有全局错误码表**，服务端返回什么 code 就原样显示在 Toast 里（见 `LoginAtPresenter#loginError`）。

## 2. 接口清单

### 2.1 账号

| 方法 | 路径 | 说明 |
|---|---|---|
| POST | `user/check_phone_available` | 手机号是否已注册 |
| POST | `user/send_code` | 发送短信验证码 |
| POST | `user/verify_code` | 校验验证码，成功返回 `verification_token` |
| POST | `user/register` | 注册（依赖上一步的 token） |
| POST | `user/login` | 登录，返回 `id` + `token` |
| GET | `user/get_token` | 用 Cookie 换新 token |
| POST | `user/set_nickname` | 修改昵称 |
| POST | `user/set_portrait_uri` | 修改头像 |
| POST | `user/change_password` | 修改密码（需旧密码） |
| POST | `user/reset_password` | 忘记密码重置（需 `verification_token`） |
| GET | `user/{userid}` | 按 id 查用户 |
| GET | `user/find/{region}/{phone}` | 按手机号查用户 |
| GET | `user/batch` | 批量查用户（群成员渲染用） |
| GET | `user/groups` | 我加入的群列表 |
| GET | `user/sync/{version}` | 增量同步好友/群关系 |
| GET | `user/get_image_token` | 取七牛上传凭证 |

### 2.2 好友关系

| 方法 | 路径 | 说明 |
|---|---|---|
| POST | `friendship/invite` | 发起好友申请，附带验证消息 |
| GET | `friendship/all` | 拉全量好友关系 |
| GET | `friendship/{userid}/profile` | 好友资料（含备注名） |
| POST | `friendship/agree` | 同意申请 |
| POST | `friendship/delete` | 删除好友 |
| POST | `friendship/set_display_name` | 设置好友备注 |
| GET | `user/blacklist` | 黑名单列表 |
| POST | `user/add_to_blacklist` / `user/remove_from_blacklist` | 加入/移出黑名单 |

### 2.3 群组

| 方法 | 路径 | 说明 |
|---|---|---|
| POST | `group/create` | 建群，传成员 id 列表与群名 |
| GET | `group/{groupId}` | 群资料 |
| GET | `group/{groupId}/members` | 群成员 |
| POST | `group/add` | 邀请成员 |
| POST | `group/kick` | 移除成员（仅群主/管理员） |
| POST | `group/rename` | 改群名 |
| POST | `group/set_portrait_uri` | 改群头像 |
| POST | `group/set_display_name` | 设置群内昵称 |
| POST | `group/quit` | 退群 |
| POST | `group/dismiss` | 解散群（仅群主） |
| POST | `group/join` | 通过二维码/邀请加入 |

### 2.4 杂项

| 方法 | 路径 | 说明 |
|---|---|---|
| GET | `misc/demo_square` | 演示用数据 |
| GET | `misc/client_version` | 版本检查 |
| GET | `downloadPic`（`@Url`） | 任意图片下载，返回 `ResponseBody` |

## 3. 典型调用链

登录：

```java
ApiRetrofit.getInstance().login(AppConst.REGION, phone, pwd)
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(resp -> {
            if (resp.getCode() == 200) {
                UserCache.save(resp.getResult().getId(), phone, resp.getResult().getToken());
                mContext.jumpToActivityAndClearTask(MainActivity.class);
            } else {
                loginError(new ServerException("登录失败:" + resp.getCode()));
            }
        }, this::loginError);
```

`AppConst.REGION` 是手机号区号（默认 `86`），服务端按 `region + phone` 定位账号。

## 4. 注意事项

- 接口都用 POST + JSON body，不要改成表单，服务端只解析 JSON
- `verification_token` 只能用一次，注册与重置密码要分别获取
- 群操作（踢人、改名、解散）的错误码没有细分，UI 上统一提示"操作失败"
- `user/sync/{version}` 需要传本地最后同步的版本号，首次同步传 `0`
- 融云相关的会话与消息**不经过这套 HTTP 接口**，见 [MESSAGE_PROTOCOL.md](MESSAGE_PROTOCOL.md)
