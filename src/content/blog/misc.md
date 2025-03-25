---
pubDatetime: 2025-01-02T13:05:09Z
modDatetime: 2025-01-02T13:10:31Z
title: 写的不知道是什么东西
featured: false
draft: false
tags:
  - DOCs
description: 
---

## Table of contents

## JRMP和RMI

JRMP是协议，RMI是一种调用方法的名称，JDK的RMI实现是使用的JRMP，weblogic的RMI实现使用的T3，JRMP其实是JDK RMI专用的，所以针对于JRMP利用就相当于是针对于JDK RMI的利用



## 对于JDK141修复

上一代是JRMP二次反序列化问题，那么JDK141修复了DGC的过滤提前，修复了反序列化失败直接清空地址，从而不能发起JRMP请求，为什么这样可以修复JRMP呢，就要详细分析一下JRMP二次流程

首先，第一次是先要去寻找一个服务端或者注册端位置去反序列化第一个点，这个位置几个可能的点，走进olddispatch之前，即注册端去改op，或者是oldDispatch之后，那只剩下DGC或者监听方法的地方

反序列化失败直接清空地址就是针对于监听位置，但是可以绕过，寻找内部本来就存在的动态代理或者。。。也会失败，因为String强制转换也会异常，那就没有方法了吗，服务端打注册端还想，客户端打服务端没了，那服务端打注册端的时候会怎么样
