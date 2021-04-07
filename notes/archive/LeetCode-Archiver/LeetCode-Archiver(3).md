---
title: "LeetCode Archiver(3)： 登录"
date: 2019-01-11T13:15:17+11:00
draft: false
categories: ["Python"]
---
## Cookie和Session

为了获取我们自己的提交记录，我们首先要进行登录的操作。但我们都知道HTTP是一种无状态的协议，它的每个请求都是独立的。无论是GET还是POST请求，都包含了处理当前这一条请求的所有信息，但它并不会涉及到状态的变化。因此，为了在无状态的HTTP协议上维护一个持久的状态，引入了Cookie和Session的概念，两者都是为了辨识用户相关信息而储存在内存或硬盘上的加密数据。

Cookie是由客户端浏览器维护的。客户端浏览器会在需要时把Cookie保存在内存中，当其再次向该域名相关的网站发出request时，浏览器会把url和Cookie一起作为request的一部分发送给服务器。服务器通过解析该Cookie来确认用户的状态，并对Cookie的内容作出相应的修改。一般来说，如果不设置过期时间，非持久Cookie会保存在内存中，浏览器关闭后就被删除了。

Session是由服务器维护的。当客户端第一次向服务器发出request后，服务器会为该客户端创建一个Session。当该客户端再次访问服务器时，服务器会根据该Session来获取相关信息。一般来说，服务器会为Seesion设置一个失效时间，当距离接收到客户端上一次发送request的时间超过这个失效时间后，服务器会主动删除Session。

两种方法都可以用来维护登录的状态。为了简便起见，本项目目前使用Session作为维护登录状态的方法。

## 获取数据

### 分析

首先我们进入[登录页面](https://leetcode.com/accounts/login/)，打开开发者工具，勾选Preserve log。为了知道在登录时浏览器向服务器提交了哪些数据，我们可以先输入一个错误的用户名和密码，便于抓包。

![7](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Use-Scrapy-to-Crawl-LeetCode/7.png)

通过分析"login/"这条request，我们可以知道我们所需要的一些关键信息，例如headers中的user-agent和referer，表单数据（form data）中的csrfmiddlewaretoken，login和password。显然，user-agent和referer我们可以直接复制下来，login和password是我们填写的用户名和密码。还有一个很陌生的csrfmiddlewaretoken。这是CSRF的中间件token，CSRF是Cross-Site Request Forgery，相关知识可以查询[跨站请求伪造的维基百科](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0)。那么现在我们就要分析这个token是从何而来。

#### 获取csrfmiddlewaretoken

我们将刚才获取到的csrfmiddlewaretoken复制下来，在开发者工具中使用搜索功能，可以发现这个csrfmiddlewaretoken出现在了登录之前的一些request对应的response中。例如在刚才打开登录页面，发送GET请求时，response的headers的set-cookie中出现了"csrftoken=..."，而这里csrftoken的值与我们需要在登录表单中提交的值完全相同。因此，我们可以通过获取刚才的response中的Cookies来获取csrfmiddlewaretoken的值。

![8](https://raw.githubusercontent.com/ZintrulCre/zintrulcre.github.io/master/data/Use-Scrapy-to-Crawl-LeetCode/8.png)

首先我们通过发送GET请求来分析一下Cookies的构成
```
login_url = "https://leetcode.com/accounts/login/"
session = requests.session()
result = session.get(login_url)
print(result)
print(type(result.cookies))
for cookie in result.cookies:
    print(type(cookie))
    print(cookie)
```

得到的结果是

- ```<Response [200]>``` 状态码200，表示请求成功
- ```<class 'requests.cookies.RequestsCookieJar'>``` cookies的类型是CookieJar
- ```<class 'http.cookiejar.Cookie'>``` 第一条cookie的类型是Cookie
- ```<Cookie__cfduid=d3e02d4309b848f9369e21671fabbce571548041181 for .leetcode.com/>``` 第一条cookie的信息
- ```<class 'http.cookiejar.Cookie'>``` 第二条cookie的类型是Cookie
- ```<Cookie csrftoken=13mQWE9tYN6g2IrlKY8oMLRc4VhVNoet4j328YdDapW2WC2nf93y5iCuzorovTDl for leetcode.com/>``` 第二条cookie的信息，也就是我们所需要的csrftoken

这样一来我们便获取到了在提交表单信息时所需要的csrfmiddlewaretoken，之后我们便可以开始着手写登录的相关代码了。顺便一提，在使用Django进行后端开发的时候自动生成的csrf token的键也叫csrfmiddlewaretoken，不知道LeetCode是不是用Django作为后端开发框架的。

### 实现

首先我们需要在爬虫开始运行之前获取登录信息，将Session作为类的成员变量保存下来，方便在获取submissions时使用。同时我们需要在与爬虫文件相同的目录下新建config.json，将自己的用户名和密码保存在该json文件里，这样就能顺利登陆了。
```
    def start_requests(self):
        self.Login()    # 登录
        questionset_url = "https://leetcode.com/api/problems/all/"
        yield scrapy.Request(url=questionset_url, callback=self.ParseQuestionSet)

    def Login(self):
    login_url = "https://leetcode.com/accounts/login/"
    login_headers = {
        "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'",
        "referer": "https://leetcode.com/accounts/login/",
        # "content-type": "multipart/form-data; boundary=----WebKitFormBoundary70YlQBtroATwu9Jx"
    }
    self.session = requests.session()
    result = self.session.get(login_url)
    file = open('./config.json', 'r')
    info = json.load(file)
    data = {"login": info["username"], "password": inf["password"],
            "csrfmiddlewaretoken": self.session.cookies['csrftoken']}
    self.session.post(login_url, data=data,headers=login_headers)
    print("login info: " + str(result))
```

注意如果在headers中填写了content-type的值，可能会产生一些奇怪的错误信息，并且后续不能正确地获取自己的submissions，只需要user_agent和refere的信息即可。

如果看到输出```login info: <Response [200]>```就代表登录成功了！

## 参考资料

<a href="https://scrapy-chs.readthedocs.io/zh_CN/0.24/" target="_blank">Scrapy官方文档</a>