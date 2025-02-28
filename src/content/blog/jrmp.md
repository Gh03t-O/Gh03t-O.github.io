---
pubDatetime: 2025-01-02T13:05:09Z
modDatetime: 2025-01-02T13:10:31Z
title: JRMP
featured: true
draft: false
tags:
  - Java
  - 反序列化
  - 利用链
description: 还没写完 先同步吧
---

## Table of contents

## JRMP

总结就是JRMP之所以可以绕过JEP290是因为在底层的JRMP通信过程中发生的序列化和反序列化没有被JEP290进行过过滤，那我们就可以考虑使用JRMP通信进行反序列化。不过反序列化是在客户端进行的，也就是说要想在服务器进行反序列化，那么只能让服务器向hacker机器发送JRMP连接请求。所以问题就变成了怎么让服务器发送JRMP请求（SSRF？），或者是使用不被JEP290拦截的可以发起JRMP请求的链子。ysoserial的JRMPClient就可以生成这种反序列化数据。

### JRMP通信过程中的触发点







## JRMPClient链子分析



