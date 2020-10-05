---
title: cookie+sessionId 和 token 登陆
date: 2020-09-08 23:44:26
author: "BerniLin21"
header-img: /images/bg1.png
tags:
---

*这里讲到的是最常见的 cookie + session 登陆和 token 登陆，其他登陆方式可能会在后续提到。。。*


## cookie + session 登陆

### cookie

- HTTP 是无状态的协议，对于事务处理没有记忆能力，每次客户端和服务端会话完成时，服务端不会保存任何会话信息。服务端无法确认当前访问者的身份信息，无法分辨上一次的请求发送者和这一次的发送者是不是同一个人

- 服务端向客户端发送一段特殊的信息，客户端以 `key: value` 的方式存存储起来，客户端再向服务端端发送请求时会带上这些信息

- 客户端请求服务端，服务端会为这次请求开辟一块内存空间，即 session 对象


### session

- session 是记录服务器和客户端会话状态的机制

- session 是基于 cookie 实现的，session 存储在服务器端，sessionId 会被存储到客户端的 cookie 中


### cookie + session 实现流程

1、当用户访问 a.com 时浏览器发送的请求携带 cookie 给服务器
2、服务器比对 cookie 中的 sessionId 和保存在服务器端的 sessionId 是否一致
3、如果一致，则身份验证成功，不一致则跳转登陆页面
4、用户在登陆页面输入账号密码
5、服务器验证账号密码无误后创建 sessionId 并保存起来
6、服务器通过 `Set-Cookie` 头信息将 sessionId 写入 cookie
7、登陆成功

具体流程如下：
![流程][1]


### cookie + session 的不足

* 如果用户多的情况下服务器需要存放大量的 sessionId ，加大了服务器的负担
* 服务器需要同步所有的 sessionId 增加运维成本
* sessionId 存放在浏览器的 cookie 中，无法避免 CSRF 攻击



## token 登陆

Token 是访问资源接口（API）时所需要的资源凭证
当第一次登录后，服务器会生成一个 Token 并返回给客户端，
客户端后续访问时，只需带上这个 Token 即可完成身份认证。

*为了解决 cookie + token 模式下的问题，可以使用 token 来实现登陆。*

### token 的实现流程

1、当用户访问 a.com 向服务器发起请求并带上 token
2、服务器验证 token，返回验证结果，成功则继续访问，失败则跳转登录页面
3、用户在登陆页面输入账号密码进行登陆
4、服务器接收到账号密码并确认无误后创建 token
5、服务器端将 Token 返回给客户端，由客户端自行保存

具体流程如下：
![流程][2]
*此处省略前端自行校验登陆状态*


### token 的特点

* token 生成后不需要存放在服务器，减少服务器的压力
* token 由前端自由保存，不必存放在 cookie ，提高安全性





[1]: /images/cookie+sessionId.png
[2]: /images/token.png
