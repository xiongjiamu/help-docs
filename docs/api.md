# API 文档

> 完整的 API 文档参考 Gitlab API docs：[https://docs.gitlab.com/13.2/ee/api/README.html](https://docs.gitlab.com/13.2/ee/api/README.html)

主要的 GitLab API 是[REST](https://en.wikipedia.org/wiki/Representational_state_transfer) API。因此，本文档假定您了解 REST 的概念及知识。

## 可用的 API 资源[](#available-api-resources "Permalink")

有关可用资源及其接口的列表，请参阅[API](https://docs.gitlab.com/13.2/ee/api/api_resources.html) 资源。

## 熟悉 GraphQL[](#road-to-graphql "Permalink")

[GitLab](graphql/index.html)中提供了[GraphQL](https://docs.gitlab.com/13.2/ee/api/graphql/index.html) ，它将允许弃用控制器特定的接口。

GraphQL 具有许多优点：

1.  我们避免维护两个不同的 API
2.  API 的调用者只能请求他们需要的东西
3.  版本已经默认

它将与当前的 v4 REST API 共存；如果我们有 v5 API，则它应该是 GraphQL 之上的兼容性层。

尽管 GraphQL 存在一些专利和许可问题，但通过 MIT 下的参考实现的重新许可以及 GraphQL 规范使用 OWF 许可证，我们已经满意得解决了这些问题。

## 兼容性指引[](#compatibility-guidelines "Permalink")

HTTP API 使用单个数字进行版本控制，当前版本为 4。此数字与[SemVer](https://semver.org/)描述的主要版本号[相同](https://semver.org/) ，这意味着向后不兼容的更改将需要更改此版本号。但是，次要版本不明确。这可以实现稳定的 API 接口，但也意味着可以将新功能以相同的版本号添加到 API。

新功能和错误修复将与新的 GitLab 于每月 22 日发布一起发布，附带的修补程序和安全版本除外。向后不兼容的更改（例如，接口删除，参数删除等）以及整个 API 版本的删除与 GitLab 本身的主要版本一起进行，两个版本之间的所有弃用和更改都应在文档中列出。 对于 v3 和 v4 之间的更改，请阅读[v3 至 v4 文档](https://docs.gitlab.com/13.2/ee/api/v3_to_v4.html)。

### 当前版本[](#current-status "Permalink")

当前仅提供 API 版本 v4，v3 版本已在[GitLab 11.0](https://gitlab.com/gitlab-org/gitlab-foss/-/issues/36819)中移除。

## 基础用法[](#basic-usage "Permalink")

API 请求应以`api`和 API 版本为前缀，API 版本在[`lib/api.rb`](https://gitlab.com/gitlab-org/gitlab/tree/master/lib/api/api.rb)定义，例如，v4 API 的根位于`/api/v4` 。

使用 cURL 的有效 API 请求的示例：

```
curl "https://codechina.csdn.net/api/v4/projects" 
```

该 API 使用 JSON 序列化数据，您无需在 API URL 的末尾指定`.json` 。

## 认证[](#authentication "Permalink")

大多数 API 请求都需要身份验证，或者仅在不提供身份验证时才返回公共数据。对于不需要的情况，将在文档中针对每个接口进行提及，例如， [`/projects/:id`接口](https://docs.gitlab.com/13.2/ee/api/projects.html#get-single-project)。

我们建议使用以下方式来进行身份验证：

1.  [Personal access tokens](/docs/user/profile/token-)

如果身份验证信息无效或被忽略，将返回一条状态码为`401`的错误消息：

```
{  "message":  "401 Unauthorized"  } 
```

### 用户访问令牌[](#personalproject-access-tokens "Permalink")

访问令牌可通过 `private_token` 参数或 `Private-Token` 头文件来进行`API`接口的验证。

在参数中使用个人访问令牌的示例：

```
curl "https://codechina.csdn.net/api/v4/projects?private_token=<your_access_token>" 
```

在头文件使用个人访问令牌的示例：

```
curl --header "Private-Token: <your_access_token>" "https://codechina.csdn.net/api/v4/projects" 
```

## 状态码[](#status-codes "Permalink")

API 可以根据上下文和操作返回不同的状态码，如果请求导致错误，则调用者可以通过状态码了解出了什么问题。

下表概述了 API 函数的一般行为：

| 请求类型 | 说明 |
| --- | --- |
| `GET` | 访问一个或多个资源，并将结果作为 JSON 返回 |
| `POST` | 如果成功创建资源，则返回`201 Created` ，然后将新创建的资源作为 JSON 返回 |
| `GET` / `PUT` | 如果成功访问或修改了资源，则返回`200 OK` ，（修改后的）结果作为 JSON 返回 |
| `DELETE` | 如果资源已成功删除，则返回`204 No Content`  |

下表显示了 API 请求可能的返回码：

| 返回值 | 说明 |
| --- | --- |
| `200 OK` | `GET` ， `PUT`或`DELETE`请求成功，资源本身作为 JSON 返回 |
| `204 No Content` | 服务器已成功满足请求，并且响应有效载荷主体中没有其他要发送的内容 |
| `201 Created` | `POST`请求成功，资源作为 JSON 返回 |
| `304 Not Modified` | 指示自上次请求以来尚未修改资源 |
| `400 Bad Request` | API 请求的必需属性丢失，例如，未提供问题标题 |
| `401 Unauthorized` | 用户未通过身份验证，有效的[用户令牌](#authentication)是必需的 |
| `403 Forbidden` | 不允许该请求，例如，不允许用户删除项目 |
| `404 Not Found` | 无法访问资源，例如，找不到资源的 ID |
| `405 Method Not Allowed` | 服务器不支持该请求 |
| `409 Conflict` | 已经存在冲突的资源，例如，使用已经存在的名称创建项目 |
| `412` | 表示请求被拒绝 `If-Unmodified-Since`尝试删除资源时提供了`If-Unmodified-Since`标头，则可能会发生这种情况 |
| `422 Unprocessable` | 无法处理该实体 |
| `500 Server Error` | 在处理请求时，服务器端出了问题 |

## 分页方法[](#pagination "Permalink")

我们支持两种分页方法：

*   基于偏移的分页，这是默认方法，并且在所有接口上都可用
*   基于键集的分页，已添加到选定的接口，但会[逐步推出](https://gitlab.com/groups/gitlab-org/-/epics/2039) 

对于大型集合，出于性能原因，我们建议键集分页（如果有）而不是偏移分页。

### 偏移分页[](#offset-based-pagination "Permalink")

有时，返回的结果将跨越许多页面。列出资源时，可以传递以下参数：

| 参数 | 说明 |
| --- | --- |
| `page` | 页码（默认： `1` ） |
| `per_page` | 每页要列出的项目数（默认： `20` ，最大： `100` ） |

在下面的示例中，我们每页列出 50 个名称空间。

```
curl --request PUT --header "PRIVATE-TOKEN: <your_access_token>" "https://codechina.csdn.net/api/v4/namespaces?per_page=50" 
```

#### 分页链接头[](#pagination-link-header "Permalink")

[链接头](https://www.w3.org/wiki/LinkHeader)随每个响应一起发送回去，它们的`rel`设置为 prev / next / first / last 并包含相关的 URL。请使用这些链接，而不是生成自己的 URL。

在下面的 cURL 调用例子中，我们限制输出到每页 3 项（ `per_page=3` ），我们要求的第二页（ `page=2`的） 评论与 ID 问题的`8`属于与 ID 项目`8` ：

```
curl --head --header "PRIVATE-TOKEN: <your_access_token>" "https://codechina.csdn.net/api/v4/projects/8/issues/8/notes?per_page=3&page=2" 
```

响应将是：

```
HTTP/1.1 200 OK
Cache-Control: no-cache
Content-Length: 1103
Content-Type: application/json
Date: Mon, 18 Jan 2020 09:43:18 GMT
Link: <https://codechina.csdn.net/api/v4/projects/8/issues/8/notes?page=1&per_page=3>; rel="prev", <https://gitlab.example.com/api/v4/projects/8/issues/8/notes?page=3&per_page=3>; rel="next", <https://gitlab.example.com/api/v4/projects/8/issues/8/notes?page=1&per_page=3>; rel="first", <https://gitlab.example.com/api/v4/projects/8/issues/8/notes?page=3&per_page=3>; rel="last"
Status: 200 OK
Vary: Origin
X-Next-Page: 3
X-Page: 2
X-Per-Page: 3
X-Prev-Page: 1
X-Request-Id: 732ad4ee-9870-4866-a199-a9db0cde3c86
X-Runtime: 0.108688
X-Total: 8
X-Total-Pages: 3 
```

#### 其他分页头[](#other-pagination-headers "Permalink")

其他分页头信息也会发送回：

| 头信息 | 说明 |
| --- | --- |
| `X-Total` | 项目总数 |
| `X-Total-Pages` | 总页数 |
| `X-Per-Page` | 每页的项目数 |
| `X-Page` | 当前页面的索引（从 1 开始） |
| `X-Next-Page` | 下一页的索引 |
| `X-Prev-Page` | 前一页的索引 |

**注意：**由于性能原因，自[GitLab 11.8 起](https://gitlab.com/gitlab-org/gitlab-foss/-/merge_requests/23931)并**在`api_kaminari_count_with_limit` 功能标志后面** ，如果资源数量大于 10,000，则`X-Total`和`X-Total-Pages`标头以及`rel="last"` `Link`将不存在响应头。

### 键集分页[](#keyset-based-pagination "Permalink")

键集分页可以更有效地检索页面，并且与基于偏移的分页相比，运行时与集合的大小无关。

此方法由以下参数控制：

| 参数 | 说明 |
| --- | --- |
| `pagination` | `keyset` （启用键集分页） |
| `per_page` | 每页要列出的项目数（默认： `20` ，最大： `100` ） |

在下面的示例中，我们每页列出 50 [个项目](/docs/user/project-) ，并按`id`升序排列。

```
curl --request GET --header "PRIVATE-TOKEN: <your_access_token>" "https://codechina.csdn.net/api/v4/projects?pagination=keyset&per_page=50&order_by=id&sort=asc" 
```

响应头包括指向下一页的链接，例如：

```
HTTP/1.1 200 OK
...
Links: <https://codechina.csdn.net/api/v4/projects?pagination=keyset&per_page=50&order_by=id&sort=asc&id_after=42>; rel="next"
Link: <https://codechina.csdn.net/api/v4/projects?pagination=keyset&per_page=50&order_by=id&sort=asc&id_after=42>; rel="next"
Status: 200 OK
... 
```

**弃用：** `Links`标题将在 GitLab 14.0 中删除，以符合[W3C `Link`规范](https://www.w3.org/wiki/LinkHeader)

指向下一页的链接包含一个附加过滤器`id_after=42` ，该过滤器不包括我们已经检索到的记录。请注意，过滤器的类型取决于所使用的`order_by`选项，我们可能有多个过滤器。

当到达集合的末尾并且没有其他要检索的记录时，将没有`Links`头，并且结果数组为空。

我们建议仅使用给定的链接来检索下一页，而不要构建自己的 URL. 除了显示的标题外，我们不公开其他分页标题。

仅针对所选资源和订购选项支持基于键集的分页：

| 资源 | 排序 |
| --- | --- |
| [Projects](/docs/user/project-) | `order_by=id` only |

## 路径参数[](#path-parameters "Permalink")

如果接口具有路径参数，则文档将在它们前面加上冒号。

例如：

```
DELETE /projects/:id/share/:group_id 
```

`:id`路径参数需要替换为项目 ID，而`:group_id`需要替换为组织的 ID，冒号`:`不应包含在内。

因此，对 ID 为`5` ，组 ID 为`17`的项目的 cURL 调用为：

```
curl --request DELETE --header "PRIVATE-TOKEN: <your_access_token>" "https://codechina.csdn.net/api/v4/projects/5/share/17" 
```

**注意：**必须遵循要求进行 URL 编码的路径参数。 如果不是，它将与 API 接口不匹配并以 404 进行响应。如果 API 前面有某些内容（例如 Apache），请确保它不会对 URL 编码的路径参数进行解码。

## 命名空间路径编码[](#namespaced-path-encoding "Permalink")

如果使用命名空间的 API 调用，请确保`NAMESPACE/PROJECT_PATH`是 URL 编码的。

例如， `/`由`%2F`表示：

```
GET /api/v4/projects/diaspora%2Fdiaspora 
```

**注意：**项目的**路径**不必与**名称**相同，可以在项目的 URL 或项目设置中的**常规>高级>更改路径**下找到项目的**路径** 。

## 文件路径、分支、tag 编码[](#file-path-branches-and-tags-name-encoding "Permalink")

如果文件路径，分支或标记包含`/` ，请确保其经过 URL 编码。

例如， `/`由`%2F`表示：

```
GET /api/v4/projects/1/repository/files/src%2FREADME-?ref=master
GET /api/v4/projects/1/branches/my%2Fbranch/commits
GET /api/v4/projects/1/repository/tags/my%2Ftag 
```

## 请求有效载荷[](#request-payload "Permalink")

API 请求可以使用作为[查询字符串](https://en.wikipedia.org/wiki/Query_string)或[有效载荷主体](https://tools.ietf.org/html/draft-ietf-httpbis-p3-payload-14#section-3.2)发送的参数。GET 请求通常发送查询字符串，而 PUT / POST 请求通常发送有效内容正文：

*   请求参数：

    ```
    curl --request POST "https://gitlab/api/v4/projects?name=<example-name>&description=<example-description>" 
    ```

*   请求有效载荷（JSON）：

    ```
    curl --request POST --header "Content-Type: application/json" --data '{"name":"<example-name>", "description":"<example-description"}' "https://gitlab/api/v4/projects" 
    ```

URL 编码的查询字符串具有长度限制，请求太大将导致`414 Request-URI Too Large`错误消息，这可以通过使用有效载荷主体来解决。

## Encoding API parameters of `array` and `hash` types[](#encoding-api-parameters-of-array-and-hash-types "Permalink")

我们可以使用`array`和`hash`类型参数调用 API，如下所示：

### `array`[](#array "Permalink")

`import_sources`是`array`类型的参数：

```
curl --request POST --header "PRIVATE-TOKEN: <your_access_token>" \
-d "import_sources[]=github" \
-d "import_sources[]=bitbucket" \
https://codechina.csdn.net/api/v4/some_endpoint 
```

### `hash`[](#hash "Permalink")

`override_params`是`hash`类型的参数：

```
curl --request POST --header "PRIVATE-TOKEN: <your_access_token>" \
--form "namespace=email" \
--form "path=impapi" \
--form "file=@/path/to/somefile.txt"
--form "override_params[visibility]=private" \
--form "override_params[some_other_param]=some_value" \
https://codechina.csdn.net/api/v4/projects/import 
```

### hash 键对[](#array-of-hashes "Permalink")

`variables`是包含哈希键/值对`[{ 'key' => 'UPLOAD_TO_S3', 'value' => 'true' }]` `array`类型的参数：

```
curl --globoff --request POST --header "PRIVATE-TOKEN: <your_access_token>" \
"https://codechina.csdn.net/api/v4/projects/169/pipeline?ref=master&variables[][key]=VAR1&variables[][value]=hello&variables[][key]=VAR2&variables[][value]=world"

curl --request POST --header "PRIVATE-TOKEN: <your_access_token>" \
--header "Content-Type: application/json" \
--data '{ "ref": "master", "variables": [ {"key": "VAR1", "value": "hello"}, {"key": "VAR2", "value": "world"} ] }' \
"https://codechina.csdn.net/api/v4/projects/169/pipeline" 
```

## `id` vs `iid`[](#id-vs-iid "Permalink")

Some resources have two similarly-named fields. For example, [issues](issues.html), [merge requests](merge_requests.html), and [project milestones](merge_requests.html). The fields are:

*   `id` ：在所有项目中唯一的 ID.
*   `iid` ：在单个项目范围内唯一的附加内部 ID.

**注意：** `iid`显示在 Web UI 中.

如果资源具有`iid`字段和`id`字段，则通常使用`iid`字段代替`id`来获取资源。

例如，假设一个`id: 42`的项目有一个`id: 46`和`iid: 5` ，在这种情况下：

*   检索问题的有效 API 调用是`GET /projects/42/issues/5`
*   检索问题的无效 API 调用是`GET /projects/42/issues/46` 

**注意：**并非所有具有`iid`字段的资源都由`iid`获取， 有关使用哪个字段的指导，请参阅特定资源的文档。

## 数据验证及错误报告[](#data-validation-and-error-reporting "Permalink")

使用 API​​时，您可能会遇到验证错误，在这种情况下，API 会以 HTTP `400`状态进行回答。

这种错误在两种情况下出现：

*   API 请求的必需属性丢失，例如，未给出问题标题
*   属性未通过验证，例如，用户描述太长

当缺少属性时，您将获得类似以下内容的信息：

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
{
 "message":"400 (Bad request) \"title\" not given"
} 
```

发生验证错误时，错误消息将有所不同。它们将包含验证错误的所有详细信息：

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
{
 "message": {
 "bio": [
 "is too long (maximum is 255 characters)"
 ]
 }
} 
```

这使错误消息更加机器可读. 格式可以描述如下：

```
{  "message":  {  "<property-name>":  [  "<error-message>",  "<error-message>",  ...  ],  "<embed-entity>":  {  "<property-name>":  [  "<error-message>",  "<error-message>",  ...  ],  }  }  } 
```

## 不存在的路由[](#unknown-route "Permalink")

当您尝试访问不存在的 API URL 时，您将收到 404 Not Found。

```
HTTP/1.1 404 Not Found
Content-Type: application/json
{
 "error": "404 Not Found"
} 
```

## Encoding `+` in ISO 8601 dates[](#encoding--in-iso-8601-dates "Permalink")

如果需要在查询参数中包含`+` ，则由于[W3 建议](http://www.w3.org/Addressing/URL/4_URI_Recommentations.html)会将`+`解释为空格，因此可能需要使用`%2B`来代替。 例如，在一个 ISO 8601 日期中，您可能希望以"山区标准时间"传递时间，例如：

```
2017-10-17T23:11:13.000+05:30 
```

The correct encoding for the query parameter would be:

```
2017-10-17T23:11:13.000%2B05:30 
```

## 客户端[](#clients "Permalink")

大多数流行的编程语言都有许多非官方的 GitLab API 客户端， 访问[GitLab 网站](https://about.gitlab.com/partners/#api-clients)以获取完整列表。

## 频率限制[](#rate-limits "Permalink")

有关速率限制设置的管理员文档，请参阅[速率限制](https://docs.gitlab.com/13.2/ee/security/rate_limits.html)， 要查找 GitLab.com 专门使用的设置，请参阅[GitLab.com 特定的速率限制](https://docs.gitlab.com/13.2/ee/user/gitlab_com/index.html#gitlabcom-specific-rate-limits) 。