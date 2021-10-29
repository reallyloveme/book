#  cookie，session，单点登录，token

### 1 cookie

 在提供标记的接口，通过HTTP返回头的Set-Cookie字段，直接种在浏览器上

浏览器发送请求的时候会直接带给接口

通过配置Domain/Path，可以设置在相同域名或者Path地址上携带该cookie

#### cookie有效期设置

Expires： 设置具体的时间，指浏览器到达指定时间就失效

Max-Age： 设置的是秒数，过了这个时间，浏览器就不保存cookie了

Max-Age优先级高于Expires，如果设置Max-Age和Expires，那么Max-Age会优先生效



#### Secure/HttpOnly

Secure是指浏览器只有在Https下，才会把cookie发送到服务器

HttpOnly是指禁止通过js脚本获取cookie，只能在请求头中进行cookie的传输



### 2 session

![img](https://gitee.com/myreally/pic/raw/master/b7add15094f3135cec700dad381a2d7f-20211017120425379.svg)

1. 浏览器登录账户密码
2. 服务端通过查询用户库校验账号密码有效性

1. 校验成功，服务器把用户登录状态存为session，生成一个sessionId
2. 通过登录接口返回，把sessionId set到cookie上

1. 浏览器请求接口的时候会把sessionId随cookie带上
2. 服务端查询session表，校验session

1. 成功后会返回结果

**session 存储方式**

1. **redis：内存型数据库，以key-value的方式存，访问快**
2. **内存：直接存在变量中，重启服务就没有了**

1. **数据库：普通数据库，性能不高**



### 3 Token

![img](https://gitee.com/myreally/pic/raw/master/58c7f4ff390371fd0578ce0d01a48b9c-20211017120217477.svg)

token流程：

1. 用户登录账号密码，服务端查询用户表，校验账号密码
2. 返回用户数据生成token，通过set cookie设置到浏览器中

1. 用户请求接口会通过cookie携带token
2. 服务端校验token的有效性，进行正常的业务接口处理



token可以通过cookie保存也可以自定义参数保存，是无状态的

token通过base64，jwt都可以生成



#### refresh token

 目的： 避免token频繁过期，用户退出登录，通过refresh token获取access token，有效期可以长一点，通过独立的服务和严格的请求方式增加安全性

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b764b256211b4ea182388fd92674fe70~tplv-k3u1fbpfcp-watermark.awebp)

refresh token流程

1. 用户登录账号密码，服务端查询用户表，校验账号密码，在认证服务生成refresh token和access token
2. 返回refresh token和access token到客户端存储起来
3. 用户进行业务处理，如果access token有效则进行接口数据处理
4. 如果access token无效，则校验refresh token是否有效，在认证服务依据refresh token生成最新的access token，进行接口操作树立
5. 如果refresh token无效，则只能退出登录

### 4. 单点登录

#### 4.1 主域名相同的单点登录

系统在同一个主域名下，利于**a.baidu.com** **b.baidu.com**则直接把cookie domain设置为主域名 **baidu.com**

![image-20211017102619815](https://gitee.com/myreally/pic/raw/master/image-20211017102619815.png)

#### 4.2 主域名不同的单点登录

例如：**a.cc.com, b.dd.com, c.ee.com**实现**一次登录，全线通用**，则需要**sso（独立的认证服务）**

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5a94e9c990d4d69a4a0845c4f8dc6a1~tplv-k3u1fbpfcp-watermark.awebp)

1. 在 SSO 域下，SSO 不是通过接口把 ticket 直接返回，而是通过一个带 code 的 URL 重定向到系统 A 的接口上，这个接口通常在 A 向 SSO 注册时约定
2. 浏览器被重定向到 A 域下，带着 code 访问了 A 的 callback 接口，callback 接口通过 code 换取 ticket
3. 这个 code 不同于 ticket，code 是一次性的，暴露在 URL 中，只为了传一下换 ticket，换完就失效
4. callback 接口拿到 ticket 后，在自己的域下 set cookie 成功
5. 在后续请求中，只需要把 cookie 中的 ticket 解析出来，去 SSO 验证就好
6. 访问 B 系统也是一样