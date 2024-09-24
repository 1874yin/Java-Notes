用户登录实现：Session + Cookie vs Token 

两种常用的登录实现方式：

- Cookie + Session
- Token 登录

### Cookie + Session

HTTP 是一种无状态的协议，每次发起请求，首先和服务端建立一个连接，请求完成后又会断开这个连接。这种方式可以节省传输时所占的资源，但又存在一个问题：所有请求都是独立的。服务器无法判断本次请求和上一次请求是否来自同一个用户，也就无法判断用户的登录状态。

为了解决 HTTP 无状态的问题，Cookie 被提出：

```
Cookie 是服务器发给客户端的一段特殊信息，以文本的形式存放在客户端，客户端每次向服务器发起请求都会带上这些信息。
```

有了 Cookie 之后，服务端就能够获取客户端传递过来的信息了，而校验信息，还需要用到 Session。

```
客户端请求服务端，服务端会为这次请求开辟一块内存空间，这个就是 Session，相当于一次客户端和服务端之间的会话。
```

#### Cookie + Session 流程

用户首次登录时：

![img](https://i-blog.csdnimg.cn/blog_migrate/6d7d582e85226da9ff7565a8c13ac5d5.png)

- 用户访问网站，输入账号密码登录
- 服务端验证账号密码，如果通过，则创建并保存 SessionId（内存、数据库、文件）
- 将 SessionId 写入 Cookie 中发送给客户端
- 登录成功

后续的访问就可以直接使用 Cookie 进行身份验证了：

![img](https://i-blog.csdnimg.cn/blog_migrate/a83b8facbf9dc9e1988f82d122b51efc.png)

- 用户访问网站，会自动带上第一次登录时写入的 Cookie（浏览器自动完成）
- 服务端比对 Cookie 中 SessionId 的值，与保存在服务器中的 SessionId 是否一致

- 一致则表明身份验证成功

Note：另一种更为简便的方法是用 JSESSIONID 为 key 将 SessionId 写入 Cookie 中，在服务端获取 session 时使用：

```java
HttpSession session = httpServeltRequest.getSession(true);
```

只要将参数设置为 true，框架便会先查找有无对应 SessionId 的 Session 对象，有则直接返回对应的 Session，否则新建一个 Session。如此便无需自己在服务端保存用户登录时对应的 SessionId。

#### Cookie + Session 存在的问题

- 服务端需要存放大量的 SessionId，会导致服务器压力过大
- 如果服务端是一个集群，为了同步登录状态，需要将 SessionId 同步到每一台机器上，增加服务器维护成本
- SessionId 放在 Cookie 中，无法避免 CSRF 攻击

### Token 登录

```
Token 是服务端生成的一串字符串，作为客户端请求的一个令牌。第一次登录后，服务器会生成一个 token 返回给客户端，客户端的后续访问只需要带上这个 token 即可完成身份验证。
```

#### Token 登录流程

首次登录时：

![img](https://i-blog.csdnimg.cn/blog_migrate/ba60c2625931d99ca487f88f579c4bfe.png)

- 访问网站，输入账号密码
- 服务端验证账号密码，如果通过，则创建 Token
- 将 Token 返回给前端，前端自由保存

后续访问页面：

![img](https://i-blog.csdnimg.cn/blog_migrate/2049a80f3ae2c13a51368f45fae1e56a.png)

- 访问页面时，带上 Token
- 服务端校验Token

#### Token 机制的特点

- 无须服务端保存 Token，不会对服务器产生压力，即使是集群，也不会增加维护成本
- Token 可以存放在前端任何地方，而不是必须放在 Cookie 中，提升页面安全性
- Token 下发后，只要在指定时间内都有效，服务端想收回 Token 的权限并不容易

#### Token 生成方式

最常用的是 JWT（Json Web Token）。

JWT 主要包含三个部分：header, payload, signature

header 包含了 JWT 使用的签名算法：

```java
header = '{"alg":"HS256", "typ":"JWT"}'
```

payload 部分表明了 JWT 的意图：

```java
payload = '{"loggedInAs":"admin","iat":"1422779638"}'
```

signature 部位为 JWT 的签名，防止 JWT 被篡改，包含两个步骤：

- 输入 base64url 编码的 header、payload，输出 unsignedToken
- 输入服务器的私钥、unsignedToken，输出签名

最后再拼接三个部分的编码后内容。

用户登录时，服务器根据签名，对 JWT 进行验证、解码、解密，校验 JWT 的有效性。