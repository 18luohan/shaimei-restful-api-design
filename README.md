# 晒美Restful API规范

* [请求](#requests)
  * [HTTP方法使用场景](#http-method-using-scene)
  * [使用一致的资源路径格式](#consistent-resource-path-formats)
  * [请求首部](#request-header)
  * [请求主体](#request-body)
  * [为了方便而支持非ID的间接引用](#support-non-id-dereferencing)
  * [最小化路径嵌套](#minimize-path-nesting)
  * [路径大小写与属性命名风格](#path-letter-case-and-attr-naming-style)
* [参考资源](#referenced-resources)


## <a id="requests">请求</a>

### <a id="http-method-using-scene">HTTP方法使用场景</a>

| HTTP METHOD | 使用场景 |
|-------------|---------|
| GET | 查找资源 |
| POST | 创建资源 |
| PUT | 修改资源 |
| PATCH | 修改资源。put与patch的不同体现在服务器处理 请求中封闭的实体以更新request uri所标示的资源 的方式。对于put请求，其中封闭的实体被认为是存储在原始服务器上的资源的一个修改版本，而客户端正在请求替换掉当前服务器上存储的版本。然而，对于patch请求，封闭的实体包含了一系列指令，它们描述当前存在于原始服务器上的某个资源应该如何被修改以生成一个新的版本。patch方法影响request uri标示的资源，并且也可能对其他资源有副作用。也就是说，应用patch方法可能会创建新资源，也可能会修改已存在资源。更多请参考：[RFC 5789](#RFC-5789) |
| DELETE | 删除资源 |

### <a id="consistent-resource-path-formats">使用一致的资源路径格式</a>

资源路径格式：

```
http(s)://:domain_name/:resources/{specific_resource}/actions/:action
```
* :domain_name 域名
* :resources 表示资源名称，使用英文复数
* :action 为个别资源指定特殊行为。首选不需要指定任何特殊行为的资源路径，即不包含/actions/:action。

示例：

```
// 查找图片
GET http://api.shaimei.com/shares/1000
// 图片格式转换
PUT http://api.shaimei.com/pictures/1001/actions/convert-format

```

### <a id="request-header">请求首部</a>

**X-Request-ID**首部：必需。客户端发起的每一个请求必须包含该首部，其值为UUID。通过在日志中记录该值，可以对每个请求进行跟踪、诊断、调试。


**Authorization**首部：绝大部分情况必需。在用户登录后，发送的每一个请求必须包含该首部，其值为服务端为每一个客户端颁发的Access Token，有一定的有效期；过期后需要通过刷新接口获取新的token。示例：Authorization: Bearer 01234567-89ab-cdef-0123-456789abcdef，其中Bearer表示此种token不会绑定其生产者和使用者，绑定token的生成者和使用者，知道token的人就能使用这个token的所有权限，就像生成token的人的权限一样。

### <a id="request-body">请求主体（Request Body）</a>

在 PUT/PATCH/POST 请求主体中使用序列化的JSON，而不是表单编码(form-encoded)的数据。例如：

```
$ curl -X POST http://api.shaimei.com/users \
    -H "Content-Type: application/json" \
    -d '{"name": "beedo","gender":"female","email":"123456@qq.com"}'

{
  "id": "01234567-89ab-cdef-0123-456789abcdef",
  "name": "beedo",
  "gender": "female",
  "email": "123456@qq.com"
}
```

### <a id="support-non-id-dereferencing">为了方便而支持非ID的间接引用</a>

在某些情况中，让最终用户提供ID来查找资源可能不方便。例如，A用户想根据名称来查找另一个用户以关注他，但是用户的唯一标示是UUID。碰到这些情况，就应该支持ID和name两种方式：

```
$ curl https://api.shaimei.com/users/{user_id_or_name}
$ curl https://api.shaimei.com/users/97addcf0-c182
$ curl https://api.shaimei.com/users/beedo
```
但也别只接受名称而排除ID。

**疑问：服务端如何区分用户发送的是ID还是name呢？仔细一想，为什么非要区分呢？查找时ID和name两个条件使用或(or)的关系不就解决了吗！?**

### <a id="minimize-path-nesting">最小化路径嵌套</a>

在带有父/子资源关系的数据模型中，路径可能会嵌套很深。例如：

```
/subjects/{subject_id}/shares/{share_id}/pictures/{picture_id}
```

首选将资源放置在根路径下，以限制嵌套深度。个人觉得这与领域驱动设计（DDD）中设计聚合（Aggregations）时控制好其大小有很大的关系。使用嵌套表示指定范围内的集合。在上述实例中，图片属于一次分享，分享又属于一个主题，可以这样表示：

```
/subjects/{subject_id}
/subjects/{subject_id}/shares
/shares/{share_id}
/shares/{share_id}/pictures
/pictures/{picture_id}
```

### <a id="path-letter-case-and-attr-naming-style">路径大小写与属性命名风格</a>

资源路径统一使用小写字母，单词之间使用连字符(-)分隔，与hostnames命名风格保持一致。例如：

```
http://open-api.shaimei.com/pictures/1001/actions/convert-format
```

不管是请求参数、请求主体中的json对象，还是响应中返回的json对象，其属性命名都采用驼峰风格，即除第一个单词首字母小写之外，其余单词首字母均大写，单词间无分隔符。例如：

```
{
	"pictureId" : "2sdfs43dfv-sdf"
}
```
## <a id="referenced-resources">参考资源</a>

<a href="http://restcookbook.com/HTTP%20Methods/put-vs-post/">restful cook book</a>
<a href="https://github.com/interagent/http-api-design">heroku http api design guide</a>
<a id="RFC-5789" href="http://tools.ietf.org/html/rfc5789">RFC 5789</a>
