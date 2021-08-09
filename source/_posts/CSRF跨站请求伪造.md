---
title: CSRF跨站请求伪造
date: 2021-01-10 13:14:48
tags:
- web
- CSRF
categories:
- web
preview: 100
---

# CSRF跨站请求伪造
CSRF(Cross Site Request Forgery)是一种攻击者盗用用户身份，在当前**已登录**的Web上执行非本意操作的攻击方法。攻击者诱导受害者进入第三方网站，在第三方网站中，向被攻击网站发送跨站请求。

## 原理

[如何防止CSRF攻击？](https://zhuanlan.zhihu.com/p/46592479)

一个典型的CSRF攻击有着如下的流程：

- 受害者登录[http://a.com](https://link.zhihu.com/?target=http%3A//a.com)，并保留了登录凭证（Cookie）。
- 攻击者引诱受害者访问了[http://b.com](https://link.zhihu.com/?target=http%3A//b.com)。
- [http://b.com](https://link.zhihu.com/?target=http%3A//b.com) 向 [http://a.com](https://link.zhihu.com/?target=http%3A//a.com) 发送了一个请求：[http://a.com/act=xx](https://link.zhihu.com/?target=http%3A//a.com/act%3Dxx)。浏览器会默认携带[http://a.com](https://link.zhihu.com/?target=http%3A//a.com)的Cookie。
- [http://a.com](https://link.zhihu.com/?target=http%3A//a.com)接收到请求后，对请求进行验证，并确认是受害者的凭证，误以为是受害者自己发送的请求。
- [http://a.com](https://link.zhihu.com/?target=http%3A//a.com)以受害者的名义执行了act=xx。
- 攻击完成，攻击者在受害者不知情的情况下，冒充受害者，让[http://a.com](https://link.zhihu.com/?target=http%3A//a.com)执行了自己定义的操作。

## **CSRF的特点**

- 攻击一般发起在第三方网站，而不是被攻击的网站。被攻击的网站无法防止攻击发生。
- 攻击利用受害者在被攻击网站的登录凭证，冒充受害者提交操作；而不是直接窃取数据。
- 整个过程攻击者并不能获取到受害者的登录凭证，仅仅是“冒用”。
- 跨站请求可以用各种方式：图片URL、超链接、CORS、Form提交等等。部分请求方式可以直接嵌入在第三方论坛、文章中，难以进行追踪。

CSRF通常是跨域的，因为外域通常更容易被攻击者掌控。但是如果本域下有容易被利用的功能，比如可以发图和链接的论坛和评论区，攻击可以直接在本域下进行，而且这种攻击更加危险。



## 防御方法

- 阻止不明外域的访问

- - 同源检测
  - Samesite Cookie

- 提交时要求附加本域才能获取的信息

- - CSRF Token
  - 双重Cookie验证

### 同源检测

#### 验证 HTTP Referer 字段

根据 HTTP 协议，在 HTTP 头中有一个字段叫 Referer，它记录了该 HTTP 请求的来源地址。
如果黑客要对银行网站实施 CSRF 攻击，他只能在他自己的网站构造请求，当用户通过黑客的网站发送请求到银行时，该请求的 Referer 是指向黑客自己的网站。因此，要防御 CSRF 攻击，银行网站只需要对于每一个转账请求验证其 Referer 值，如果是以 bank.example 开头的域名，则说明该请求是来自银行网站自己的请求，是合法的。如果 Referer 是其他网站的话，则有可能是黑客的 CSRF 攻击，拒绝该请求。

问题在于这种方法把安全性都依赖于浏览器来保障，从理论上来讲，这样并不安全。

### CSRF Token
前面讲到CSRF的另一个特征是，攻击者无法直接窃取到用户的信息（Cookie，Header，网站内容等），仅仅是冒用Cookie中的信息。

而CSRF攻击之所以能够成功，是因为服务器误把攻击者发送的请求当成了用户自己的请求。那么我们可以要求所有的用户请求都携带一个CSRF攻击者无法获取到的Token。服务器通过校验请求是否携带正确的Token，来把正常的请求和攻击的请求区分开，也可以防范CSRF的攻击。

#### 原理
CSRF Token的防护策略分为三个步骤：

1. 将CSRF Token输出到页面中

首先，用户打开页面的时候，服务器需要给这个用户生成一个Token，该Token通过加密算法对数据进行加密，一般Token都包括随机字符串和时间戳的组合，显然在提交时Token**不能再放在Cookie中**了，否则又会被攻击者冒用。因此，为了安全起见Token最好还是**存在服务器的Session**中，之后在每次页面加载时，使用JS遍历整个DOM树，对于DOM中所有的a和form标签后加入Token。这样可以解决大部分的请求，但是对于在页面加载之后**动态生成**的HTML代码，这种方法就没有作用，还需要程序员在编码时手动添加Token。

2. 页面提交的请求携带这个Token

对于GET请求，Token将附在请求地址之后，这样URL 就变成 http://url?csrftoken=tokenvalue。 而对于 POST 请求来说，要在 form 的最后加上：
`<input type=”hidden” name=”csrftoken” value=”tokenvalue”/>`
这样，就把Token以参数的形式加入请求了。

3. 服务器验证Token是否正确

当用户从客户端得到了Token，再次提交给服务器的时候，服务器需要判断Token的有效性，验证过程是先解密Token，对比加密字符串以及时间戳，如果加密字符串一致且时间未过期，那么这个Token就是有效的。

### JWT

[[如何通过JWT防御CSRF](https://segmentfault.com/a/1190000003716037)](https://segmentfault.com/a/1190000003716037)

## 参考
[Origin](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Origin)
[Referer](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Referer)
[CSRF 攻击的应对之道](https://developer.ibm.com/zh/articles/1102-niugang-csrf/)
[CSRF JWT](https://segmentfault.com/a/1190000003716037)
[如何在logout时销毁JWT--在client端删除](https://stackoverflow.com/questions/37959945/how-to-destroy-jwt-tokens-on-logout)