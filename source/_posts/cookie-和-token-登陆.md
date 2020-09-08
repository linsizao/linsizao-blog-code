---
title: cookie 和 token 登陆
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

