---
title: "计算机网络 HTTP 协议"
date: 2015-07-05T08:57:52+10:00
draft: true
categories: ["Computer Science"]
---

# 计算机网络

## HTTP 协议

### 报文

1. 请求报文
    - request line
    - request headers
    - request body

![Request Datagram](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Computer-Science/Request%20Datagram.png)

2. 响应报文
    - status line
    - response headers
    - response body

![Response Datagram](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Computer-Science/Response%20Datagram.png)

### HTTP 方法

1. GET：获取资源
2. POST：传输数据
3. PUT：上传文件
4. DELETE：删除文件
5. OPTIONS：查询支持的方法
6. CONNECT：在与代理服务器通信时建立隧道
7. TRACE：追踪路径
8. PATCH：修改资源

### HTTP 状态码

| 状态码 | 类别 | 含义 |
|---|---|---|
| 1xx | Informational 信息状态码 | 正在处理请求 |
| 2xx | Success 成功状态码 | 正常地处理了请求 |
| 3xx | Redirection 重定向状态码 | 请求需要进行附加操作 |
| 4xx | Client Error 客户端错误状态码 | 客户端请求无法被处理 |
| 5xx | Server Error | 服务器处理请求出错 |

### 安全性

- HTTP 所封装的信息是明文的，通过抓包工具可以分析其消息内容，内容可能会被窃听
- 不验证通信方的身份，通信方的身份有可能遭遇伪装
- 无法证明报文的完整性，报文有可能遭篡改

### Cookie 和 Session

#### Cookie

- HTTP 协议是无状态的，为了能够保存状态信息，HTTP/1.1 引入了 Cookie
- Cookie 是服务器发送到用户浏览器并保存在本地的数据；当浏览器再次向相同服务器发起请求时会附带上 Cookie 信息，用于告知服务端两个请求来自同一浏览器
- 附带 Cookie 数据会带来额外的性能开销，尤其是在移动环境下

1. 用途
    - 保存会话状态
    - 保存个性化设置
    - 浏览器行为跟踪

2. 分类
    - 会话期 Cookie：仅在会话期内有效，浏览器关闭后被自动删除
    - 持久性 Cookie：指定过期时间 Expires 或有效期max-age 的 Cookie

#### Session

- Session 存储在服务器端，更加安全
- Session 可以存储在服务器上的文件、数据库或者内存中

1. Session 流程
    - 用户登录时，将包含用户名和密码的表单放入 HTTP 请求报文中
    - 服务器验证用户名和密码，如果正确则把用户信息存储到 Session 中，其 key 为 Session ID
    - 服务器返回的响应报文的 Set-Cookie 的首部字段包含了 Session ID，客户端收到响应报文之后将该 Cookie 值存入浏览器中
    - 客户端再次向服务器发送请求时会包含 Cookie 值，服务器收到请求之后取出 Session ID，根据 Session ID 取出对应的用户信息，继续操作

2. 安全性
    - 产生不容易被猜到的 Session ID
    - 经常重新生成 Session ID
    - 在对安全性要求极高的场景下，除了使用 Session 管理用户状态，还需要对用户进行重新验证

#### 对比

- Cookie 只能存储 ASCII 码字符串，而 Session 可以存储任何类型的数据，因此在考虑数据复杂性时首选 Session
- Cookie 存储在浏览器中，安全性较低；可以通过将 Cookie 进行加密并在服务器进行解密来提高安全性
- 如果用户所有的信息都存储在 Session 中，那么开销非常大；不建议将所有的用户信息都存储到 Session 中

## HTTPS 协议

- HTTPS 在 HTTP 的基础上使用了隧道进行通信：让 HTTP 先和 TLS（Transmission Layer Security）/SSL（Secure Sockets Layer）通信，再由 SSL 和 TCP 通信
- 通过使用 SSL，HTTPS 具有了加密（防窃听）、认证（防伪装）和完整性保护（防篡改）的功能

### 加密

1. 对称加密：加密和解密使用同一密钥
    - 优点：运算速度快
    - 缺点：无法安全地将密钥传输给通信方

2. 非对称加密：加密和解密使用不同的密钥
    - 所有人都可以获得公钥
    - 可以用于签名
    - 优点：传输公钥更加安全
    - 缺点：运算速度慢

3. 混合加密
    - 使用非对称密钥加密用于传输对称密钥来保证传输过程的安全性，之后再使用对称密钥进行通信来保证通信过程的效率

### 认证

- 使用证书来对通信方进行认证；数字证书认证机构（CA，Certificate Authority）是客户端与服务器双方都可信赖的第三方机构
- 服务器的运营人员向 CA 提出公开密钥的申请，CA 在判明提出申请者的身份之后，对已申请的公开密钥做数字签名，然后分配这个已签名的公开密钥，并将该公开密钥放入公开密钥证书后绑定在一起
- 进行 HTTPS 通信时，服务器把证书发送给客户端；客户端取得其中的公开密钥之后，先使用数字签名进行验证，如果验证通过，就可以开始通信了

### 缺点

- 因为需要进行加密解密，因此速度会更慢
- 需要支付证书授权的费用

### 常见问题

1. 对于使用 https 协议的网站，在浏览器上输入 http 会返回什么状态码

    - 307 Temporary Redirect
