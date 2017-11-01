FORMAT: 1A
HOST: http://api.iflytransporter.com/v1/

# Transporter

API与用户的通信协议，总是使用HTTPS协议

## HTTP动词
对于资源的具体操作类型，由HTTP动词表示

常用的HTTP动词有下面几个（括号里是对应的SQL命令）
- GET（SELECT）：从服务器取出资源（一项或多项）
- POST（CREATE）：在服务器新建一个资源
- PUT（UPDATE）：在服务器更新资源（客户端提供改变后的完整资源）
- DELETE（DELETE）：从服务器删除资源。

## 状态码
服务器向用户返回的状态码提示信息，常见的有以下一些（方括号中是该状态码对应的HTTP动词）

- 200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）
- 201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
- 202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
- 204 NO CONTENT - [DELETE]：用户删除数据成功。
- 400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
- 401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
- 403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
- 404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
- 406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
- 410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
- 422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
- 500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
## 错误处理
如果状态码不是2xx，就应该向用户返回出错信息。一般来说，返回的信息中将error作为键名，出错信息作为键值即可。
```json
 { "error" : "Invalid API key" }
```
## 鉴权
JSON Web Token (JWT)是一种开放标准（RFC 7519）（译者注：该RFC标准比较通俗易懂，建议进一步阅读），其中定义了一种紧凑 (compact) 且自包含(self-contained)（译者注：指的是在payload里包含更多的信息）的方式用于以JSON对象的形式在多方之间传递信息。信息可以被核实和信任，因为它经过了数字签名。JWT既可以使用密钥（采用HMAC算法），也可以使用公私钥（采用RSA算法）进行签名。
- 紧凑的（compact）：因为JWT更短小，所以它可以通过URL、POST参数，或存于HTTP头部发送。此外，体积小意味着传输更快。
- 自包含（self-contained）：有效载荷（payload）（译者注：通俗来讲，就是通信中实际需要传输的内容）中包含了与用户相关的所有必需信息，避免多次进行数据库查询。
### 以下是JSON Web Token的一些有效应用场景：
- 身份验证（Authentication）：这是使用JWT的最主要场景。一旦用户登录成功，每次后续的请求头包含了JWT，允许用户使用获得的令牌（token）来访问路由（routes）、服务和资源。单点登录（Single Sign On）是如今广泛使用JWT实现的一个特性，因为它具有短有效负载、便于跨域等优点。
- 信息交换：JSON Web Token是一种在多方间安全传输信息的办法。由于经过了签名（signed），例如使用公私密钥对，因此你可以确信发送方确实是他自己。另外，由于签名是由头部和有效载荷进行计算而得到的，所以你可以验证信息是否被篡改过。
### JSON Web Token的结构
JSON Web Token包含了3个部分，使用点（.）进行分隔，它们是：
- 头部（Header）
- 有效负荷（Payload）
- 签名（Signature）
因此，一个JWT通常长这样：xxxxx.yyyy.zzzz
### 工作方式
在身份验证中，当用户成功地使用它们的密码登录后，一个JSON Web Token就会被服务器返回，且必须存储于客户端本地，而不是传统做法中在服务器创建一个session，并返回一个cookie。
无论何时用户想要访问一个受保护的路由或资源，客户端应该随请求一起发送JWT。JWT通常在 Authentication HTTP头部中，使用 Bearer 格式（schema）存放。HTTP头部的内容可能长这样：
> Authorization: Bearer <token>
这是一个无状态（stateless）的认证机制，因为用户状态并未存放在服务器内存中。服务器的受保护路由会在Authentication头部中检查是否存在合法的JWT，如果存在，用户就被允许访问受保护的资源。JWT是自包含的，所以相关的信息都在里面，减少了多次查询数据库的必要。
这让你充分可以依赖无状态的数据API，甚至向下游服务发出请求。哪个域名提供API无所谓，所以跨域资源共享（CORS，Cross-Origin Resource Sharing）不会引起麻烦
下图展示了这个过程：
![](https://cloud.githubusercontent.com/assets/4011348/17654120/15ae01fa-62d2-11e6-8ff4-b34005639fee.png)

## 接口数据响应格式
如果直接使用HTTP Status Code，则直接返回JSON数据给调用者，如果不使用HTTP Status Code,请参考以下个格式，统一每个接口的数据结构
```json
{
    "code":0,
    "msg": "success",
    "data": { "key1": "value1", "key2": "value2" }
}
```

- code: 返回码，0表示成功，非0表示各种不同的错误
- msg: 描述信息，成功时为"success"，错误时则是错误信息
- data: 成功时返回的数据，类型为对象或数组，错误时这个字段可以省略

