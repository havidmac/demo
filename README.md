## RESTful API 设计规范

本文作为Multiable `RESTful api 设计规范`的草稿 ，欢迎大家补充!

## 内容概要

  * [RESTful API 设计规范](#restful-api-设计规范)
  * [关于「能愿动词」的使用](#关于能愿动词的使用)
  * [Protocol](#protocol)
  * [API Root URL](#api-root-url)
  * [Versioning](#versioning)
  * [HTTP Method](#http-method)
  * [Endpoints](#endpoints)
  * [URL Best Practices](#url-best-practices)
  * [Filtering](#filtering)
  * [Authentication](#authentication)
  * [Response](#response)

## 关于「能愿动词」的使用

为了避免歧义，文档大量使用了「能愿动词」，对应的解释如下：

* `必须 (MUST)`：绝对，严格遵循，请照做，无条件遵守；
* `一定不可 (MUST NOT)`：禁令，严令禁止；
* `应该 (SHOULD)` ：强烈建议这样做，但是不强求；
* `不该 (SHOULD NOT)`：强烈不建议这样做，但是不强求；
* `可以 (MAY)` 和 `可选 (OPTIONAL)` ：选择性高一点，在这个文档内，此词语使用较少；

> 参见：[RFC 2119](http://www.ietf.org/rfc/rfc2119.txt)

## Protocol

客户端在通过 `API` 与后端服务通信的过程中，`应该` 使用 `HTTPS` 协议。

## API Root URL

`API` 的根入口点应尽可能保持简单，这里有两个常见的例子:

* api.example.com/*
* example.com/api/*

> 如果你的应用很庞大或者你预计它将会变的很庞大，那 `应该` 将 `API` 放到子域下（`api.example.com`）。这种做法可以保持灵活性。

## Versioning

所有的`API` `必须` 保持向后兼容，你 `必须` 在引入新版本 `API` 的同时确保旧版本 `API` 仍然可用。所以 `应该` 为其提供版本支持。

建议的作法是在 URL 中嵌入版本编号 v1,v2...,比如

```bash
api.example.com/v1/*
```

## HTTP Method

对于资源的具体操作类型，由 `HTTP` 动词表示。常用的 `HTTP` 动词有下面五个（括号里是对应的 `SQL` 命令）。

* GET（SELECT）：从服务器取出资源（一项或多项）。
* POST（CREATE）：在服务器新建一个资源。
* PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源，整个替换）。
* PATCH（UPDATE）：在服务器更新资源（客户端提供改变的属性，部分属性更新）。
* DELETE（DELETE）：从服务器删除资源。

其中

1. 创建新的资源 `必须` 使用 `POST` 方法
2. 更新资源 `应该` 使用 `PUT` 方法
3. 删除资源 `必须` 用 `DELETE` 方法
4. 获取资源信息 `必须` 使用 `GET` 方法  

针对每一个端点来说，下面列出可行的 `HTTP` 动词和端点的组合

| 请求方法 | URL | 描述 |
| ---------- | --- | --- |
| GET | /employees | 列出所有的员工 |
| POST | /employees | 新增一个新的员工 |
| GET | /employees/{empId} | 获取指定员工详情 |
| PUT | /employees/{empId} | 更新指定员工(整个更新) |
| PATCH | /employees/{empId} | 更新指定员工(部分字段更新) |
| DELETE | /employees/{empId} | 删除指定员工 |
| GET | /employees/{empId}/managers | 检索指定员工下的负责人列表 |

> 超出 `Restful` 端点的，`应该` 模仿上表的方式来定义端点。

## Endpoints

端点就是指向特定资源或资源集合的 `URL`。在端点的设计中，你 `必须` 遵守下列约定：

* URL 的命名 `必须` 全部小写
* URL 中资源（`resource`）的命名 `必须` 是名词，并且 `必须` 是复数形式
* `必须` 优先使用 `Restful` 类型的 URL
* URL `必须` 是易读的
* URL `一定不可` 暴露服务器架构

来看一些**反例**

* https://api.example.com/getAllEmployees
* https://api.example.com/getAllExternalEmployees
* https://api.example.com/createEmployee
* https://api.example.com/updateEmployee
* https://api.example.com/internalAndSeniorEmployees

再来看一些**正例**

* GET  https://api.example.com/employees
* GET  https://api.example.com/employees?state=external
* POST https://api.example.com/employees
* PUT  https://api.example.com/employees/56
* GET  https://api.example.com/employees?state=internal&title=senior

## URL Best Practices

**避免多级URL**

常见的情况是，资源需要多级分类，因此很容易写出多级的 URL，比如获取某个员工的某一类配假信息:
```bash
GET https://api.example.com/employees/12/leavetypes/2
```
上面这种做法不利于扩展，更好的做法是，除了第一级，其他级别都用查询字符串表达
```bash
GET  https://api.example.com/employees/12?leavetypes=2
```
下面是另外一个例子，查询已退休的员工
```bash
GET https://api.example.com/employees/retired
```
不如
```bash
GET https://api.example.com/employees?retired=true
```
**对Action构建URL**

有时,对API调用的响应不涉及资源(如计算，转换或转换),比如下面的例子不涉及到具体的资源,在命名URL时应该使用动词
```bash
//Reading
GET /translate?from=de_DE&to=en_US&text=Hallo
GET /calculate?para2=23&para2=432

//Trigger an operation that changes the server-side state
POST /restartServer
//no body

POST /banUserFromChannel
{ "user": "123", "channel": "serious-chat-channel" }
```
其它的解决方法供参考的还有:
1. 如果Action是不带参数的，比如发布一个流程，实质是映射到流程中的一个字段```published```,那么可以将action这种形式等价为对```流程```这种资源的```PATCH```动作，部分更新```published```字段

2. 将action视为一种子资源(sub-resource),那么上面的例子,发布流程```PUT /wftpls/:id/published```,取消发布为```DELETE /wftpls/:id/published```

注意: 在实际当中，RESTful API Design并不能覆盖所有的情况，以实质重于形式考虑，使用动词命名的方法可能是更好的选择.


## Filtering

> 如果记录数量很多，服务器不可能都将它们返回给用户。API `应该` 提供参数，过滤返回结果。下面是一些常见的参数。

* ?limit=10：指定返回记录的数量
* ?offset=10：指定返回记录的开始位置。
* ?page=2&per_page=100：指定第几页，以及每页的记录数。
* ?sortby=name&order=asc：指定返回结果按照哪个属性排序，以及排序顺序。
* ?animal_type_id=1：指定筛选条件

所有 `URL` 参数 `必须` 是全小写，`必须` 使用下划线类型的参数形式。

> 分页参数 `必须` 固定为 `page`、`per_page`

经常使用的、复杂的查询 `应该` 标签化，降低维护成本。如

```bash
GET /trades?status=closed&sort=sortby=name&order=asc

# 可为其定制快捷方式
GET /trades/recently_closed
```

## Authentication

`应该` 使用 `OAuth2.0` 的方式为 API 调用者提供登录认证。`必须` 先通过登录接口获取 `Access Token` 后再通过该 `token` 调用需要身份认证的 `API`。

客户端在获得 `access token` 的同时 `必须` 在响应中包含一个名为 `expires_in` 的数据，它表示当前获得的 `token` 会在多少 `秒` 后失效。

```json
{
    "access_token": "token....",
    "token_type": "Bearer",
    "expires_in": 3600
}
```

客户端在请求需要认证的 `API` 时，`必须` 在请求头 `Authorization` 中带上 `access_token`。

```bash
Authorization: Bearer token...
```

当超过指定的秒数后，`access token` 就会过期，再次用过期/或无效的 `token` 访问时，服务端 `应该` 返回 `invalid_token` 的错误或 `401` 错误码。

```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json
Cache-Control: no-store
Pragma: no-cache

{
    "error": "invalid_token"
}
```

## Response

所有的 `API` 响应，`必须` 遵守 `HTTP` 设计规范，`必须` 选择合适的 `HTTP` 状态码。`一定不可`采用所有接口都返回状态码为 `200`，然后在Body中才定义成功或者失败的方式 ，下面是一个糟糕的例子：
```http
HTTP/1.1 200 ok
Content-Type: application/json
Server: example.com

{
    "code": 0,
    "msg": "success",
    "data": {
        "username": "username"
    }
}
```
以及
```http
HTTP/1.1 200 ok
Content-Type: application/json
Server: example.com

{
    "code": -1,
    "msg": "该活动不存在",
}
```

下表列举了常见的 `HTTP` 状态码

| 状态码 | 描述 |
| ---------- | --- |
| 2xx | 请求已成功，请求所希望的响应头或数据体将随此响应返回 |
| 4xx | 客户端原因引起的错误 |
| 5xx | 服务端原因引起的错误 |

### 200 ok

`200` 状态码是最常见的 `HTTP` 状态码，在所有 **成功** 的 `GET` 请求中，`必须` 返回此状态码。

正确示例：

1、获取单个资源详情

```json
{
    "id": 1,
    "username": "godruoyi",
    "age": 88,
}
```

2、获取资源集合

```json
[
    {
        "id": 1,
        "username": "godruoyi",
        "age": 88,
    },
    {
        "id": 2,
        "username": "foo",
        "age": 88,
    }
]
```

3、额外的媒体信息

```json
{
    "data": [
        {
            "id": 1,
            "avatar": "https://lorempixel.com/640/480/?32556",
            "nickname": "fwest",
            "last_logined_time": "2018-05-29 04:56:43",
            "has_registed": true
        },
        {
            "id": 2,
            "avatar": "https://lorempixel.com/640/480/?86144",
            "nickname": "zschowalter",
            "last_logined_time": "2018-06-16 15:18:34",
            "has_registed": true
        }
    ],
    "meta": {
        "pagination": {
            "total": 101,
            "count": 2,
            "per_page": 2,
            "current_page": 1,
            "total_pages": 51,
            "links": {
                "next": "http://api.example.com?page=2"
            }
        }
    }
}
```
> 其中，分页和其他额外的媒体信息，必须放到 `meta` 字段中。

4、返回关联信息，我们假定员工有一个管理员和若干个团队成员
```json
{
  "data": [
    { 
      "id": 1, 
      "name": "Larry",
      "relationships": {
        "manager": "http://api.example.com/employees/1/manager",
        "teamMembers": [ 
          "http://api.example.com/employees/12",
          "http://api.example.com/employees/13"
        ]
        //or "teamMembers": "http://api.example.com/employees/1/teamMembers"
      }
    }
  ]
}
```
或者
```json
{
  "data": [
    { 
      "id": 1, 
      "name": "Larry",
      "relationships": {
        "manager":  5 , 
        "teamMembers": [ 12, 13 ]
      }
    }
  ],
  "included": {
    "manager": {
      "id": 5, 
      "name": "Kevin"
    },
    "teamMembers": [
      { "id": 12, "name": "Albert" }
      , { "id": 13, "name": "Tom" }
    ]
  }
}
```
### 201 Created

当服务器创建数据成功时，`应该` 返回此状态码。常见的应用场景是使用 `POST` 提交用户信息，如：

* 添加了新用户
* 上传了图片
* 创建了新活动

等，都可以返回 `201` 状态码。需要注意的是，你可以选择在用户创建成功后返回新用户的数据

```http
HTTP/1.1 201 Created
Server: nginx/1.11.9
Content-Type: application/json
Transfer-Encoding: chunked
Date: Sun, 24 Jun 2018 09:13:40 GMT
Connection: keep-alive

{
    "id": 1,
    "avatar": "https:\/\/lorempixel.com\/640\/480\/?32556",
    "nickname": "fwest",
    "last_logined_time": "2018-05-29 04:56:43",
    "created_at": "2018-06-16 17:55:55",
    "updated_at": "2018-06-16 17:55:55"
}
```

也可以返回一个响应实体为空的 `HTTP Response` 如：

```http
HTTP/1.1 201 Created
Server: nginx/1.11.9
Content-Type: text/html; charset=UTF-8
Transfer-Encoding: chunked
Date: Sun, 24 Jun 2018 09:12:20 GMT
Connection: keep-alive
```

> 这里我们 `应该` 采用第二种方式，因为大多数情况下，客户端只需要知道该请求操作成功与否，并不需要返回新资源的信息。

### 错误处理

当 `API` 发生错误时，`必须` 返回出错时的详细信息。我们将信息直接放入响应实体中；

```http
HTTP/1.1 401 Unauthorized
Server: nginx/1.11.9
Content-Type: application/json
Transfer-Encoding: chunked
Cache-Control: no-cache, private
Date: Sun, 24 Jun 2018 10:02:59 GMT
Connection: keep-alive

{"error_code":40100,"message":"Unauthorized"}
```

错误格式 `应该` 满足如下格式：

```json
{
    "message": "您查找的资源不存在",
    "error_code": 404001
}
```

其中错误码（`error_code`）`必须` 和 `HTTP` 状态码对应，也方便错误码归类，如：

```http
HTTP/1.1 429 Too Many Requests
Server: nginx/1.11.9
Content-Type: application/json
Transfer-Encoding: chunked
Cache-Control: no-cache, private
Date: Sun, 24 Jun 2018 10:15:52 GMT
Connection: keep-alive

{"error_code":429001,"message":"你操作太频繁了"}
```

```http
HTTP/1.1 403 Forbidden
Server: nginx/1.11.9
Content-Type: application/json
Transfer-Encoding: chunked
Cache-Control: no-cache, private
Date: Sun, 24 Jun 2018 10:19:27 GMT
Connection: keep-alive

{"error_code":403002,"message":"用户已禁用"}
```

`应该` 在返回的错误信息中，同时包含面向开发者和面向用户的提示信息，前者可方便开发人员调试，后者可直接展示给终端用户查看如：

```json
{
    "message": "直接展示给终端用户的错误信息",
    "error_code": "业务错误码",
    "error": "供开发者查看的错误信息",
    "debug": [
        "错误堆栈，必须开启 debug 才存在"
    ]
}
```

下面详细列举了各种情况 API 的返回说明。

### 401 Unauthorized

该状态码表示当前请求需要身份认证，以下情况都 `必须` 返回该状态码。

* 未认证用户访问需要认证的 API
* access_token 无效/过期

> 客户端在收到 `401` 响应后，都 `应该` 提示用户进行下一步的登录操作。

```http
HTTP/1.1 401 Unauthorized
Server: nginx/1.11.9
Content-Type: application/json
Transfer-Encoding: chunked
WWW-Authenticate: JWTAuth
Cache-Control: no-cache, private
Date: Sun, 24 Jun 2018 13:17:02 GMT
Connection: keep-alive

{"message":"Token Signature could not be verified.","error_code": "40100"}
```

### 403 Forbidden

该状态码可以简单的理解为没有权限访问该请求，服务器收到请求但拒绝提供服务。

如当普通用户请求操作管理员用户时，`必须` 返回该状态码。

```http
HTTP/1.1 403 Forbidden
Server: nginx/1.11.9
Content-Type: application/json
Transfer-Encoding: chunked
Cache-Control: no-cache, private
Date: Sun, 24 Jun 2018 13:05:34 GMT
Connection: keep-alive

{"error_code":40301,"message":"权限不足"}
```

### 404 Not Found

该状态码表示用户请求的资源不存在，如

* 获取不存在的用户信息 （get /users/9999999）
* 访问不存在的端点

都 `必须` 返回该状态码，若该资源已永久不存在，则 `应该` 返回 `410` 响应。

### 405 Method Not Allowed

当客户端使用的 `HTTP` 请求方法不被服务器允许时，`必须` 返回该状态码。

> 如客户端调用了 `POST` 方法来访问只支持 GET 方法的 API

该响应 `必须` 返回一个 `Allow` 头信息用以表示出当前资源能够接受的请求方法的列表。

```http
HTTP/1.1 405 Method Not Allowed
Server: nginx/1.11.9
Content-Type: application/json
Transfer-Encoding: chunked
Allow: GET, HEAD
Cache-Control: no-cache, private
Date: Sun, 24 Jun 2018 12:30:57 GMT
Connection: keep-alive

{"message":"405 Method Not Allowed","error_code": 40500}
```

### 406 Not Acceptable

`API` 在不支持客户端指定的数据格式时，应该返回此状态码。如支持 `JSON` 和 `XML` 输出的 `API` 被指定返回 `YAML` 格式的数据时。

> Http 协议一般通过请求首部的 Accept 来指定数据格式

### 408 Request Timeout

客户端请求超时时 `必须` 返回该状态码，需要注意的时，该状态码表示 **客户端请求超时**，在涉及第三方 `API` 调用超时时，`一定不可` 返回该状态码。

### 410 Gone

和 `404` 类似，该状态码也表示请求的资源不存在，只是 `410` 状态码进一步表示所请求的资源已不存在，并且未来也不会存在。在收到 `410` 状态码后，客户端 `应该` 停止再次请求该资源。

### 500 Internal Server Error

该状态码 `必须` 在服务器出错时抛出，对于所有的 `500` 错误，都 `应该` 提供完整的错误信息支持，也方便跟踪调试。

### 503 Service Unavailable

该状态码表示服务器暂时处理不可用状态，当服务器需要维护或第三方 `API` 请求超时/不可达时，都 `应该` 返回该状态码，其中若是主动关闭 API 服务，`应该 `在返回的响应首部加上 `Retry-After` 头部，表示多少秒后可以再次访问。

```http
HTTP/1.1 503 Service Unavailable
Server: nginx/1.11.9
Content-Type: application/json
Transfer-Encoding: chunked
Cache-Control: no-cache, private
Date: Sun, 24 Jun 2018 10:56:20 GMT
Retry-After: 60
Connection: keep-alive

{"error_code":50300,"message":"服务维护中"}
```

