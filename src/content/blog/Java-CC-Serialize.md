---
pubDatetime: 2025-01-02T13:05:09Z
modDatetime: 2025-01-02T13:10:31Z
title: JavaCC链反序列化全总结
featured: true
draft: false
tags:
  - Java
  - 反序列化
  - 利用链
description: CC1,CC6,CC3,CC5,CC7,CC2,CC4,利用链全总结
---

第一次接触Java反序列化是通过b站的白日梦组长（好久不更新了），强烈推荐一下
了解CC之前建议从URLDNS的链子开始，简短
因为更多的是总结复现，所以大部分是正推。

对Java反序列化利用链构造的一些理解：
* 链子包含起始点，目的点，和中间过程(好像污点分析)
* Java反序列化利用链的构造过程更像是一个倒推的过程
* 首先找到目的点（即可以RCE的地方），然后找那个位置可以调用这个点
* 一点点的向上追，直接找到readObject

## Table of contents

##  CC1
CC1来说有两个版本，一个是TransformedMap，一个是LazyMap，两个版本的链子差别不大，看下方详解

jdk版本：8u65
jdk推荐下载地址：http://www.codebaoku.com/jdk/jdk-index.html
CC版本：3.2.1
### TransformedMap版本

涉及到Transformer修饰Map和几个Transformer
#### Transformer

关于Transformer，GPT给出的解释是 Transform 是 Commons Collections 中用于对象转换的核心概念。

CC需要关注Transformer类的transform方法，整个过程中对transform方法的调用是重点

1. InvokerTransformer的Transform
![alt](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FOGphWVFc2xi7SLrwHeD2%2Fuploads%2FgxoVO32tnjN92ZWBPzP7%2F%E5%9B%BE%E7%89%87.png?alt=media)

关于反射的知识点就不做讲解了，执行input对象的iMethodName方法

2. ConstantTransformer
返回iConstant
![alt](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FOGphWVFc2xi7SLrwHeD2%2Fuploads%2FgxoVO32tnjN92ZWBPzP7%2F%E5%9B%BE%E7%89%87.png?alt=media)
这个iConstant是ConstantTransformer初始化时传入的

    ChainedTransformer



    ChainedTransformer里边包含的是一个Transformer数组，它的transform就是用前一个（第一个除外）transformer的transform方法作为下一个调用的参数

