---
description: 不对原理过多的动态调试讲解，主要是针对于JDK版本修复讲解
---

# RMI攻防历史

通信过程，代码动态调试博客推荐

{% embed url="https://xz.aliyun.com/t/8644" %}

对于高版本的RMI的分析，我们主要看低版本中哪个位置代码修改了，看怎么绕过这个位置



## JDK<8u121(JEP290之前)

这一段没有任何过滤，所以本节会介绍JDK RMI过程中所有可能利用到的点

在这一阶段中，并没有限制注册端和服务端必须在同一台机器上，所以可以在任意机器进行bind,rebind操作

### 特殊的对于客户端

不论是客户端和注册中心通信过程还是客户端和服务端的通信过程中，客户端的某一个请求都是由executeCall发起的，而这个函数中有可以利用的readObject的位置，网上常见的通过JRMP二次反序列化的第二次反序列化的点就是这里

```
sun.rmi.transport.StreamRemoteCall#executeCall
```

```java
case TransportConstants.ExceptionalReturn:
    Object ex;
    try {
        ex = in.readObject();
    } catch (Exception e) {
        throw new UnmarshalException("Error unmarshaling return", e);
    }
```

截取了部分

### 客户端/服务端攻击注册中心

客户端和服务端攻击注册中心其实是一样的，本质上访问的是相同的skel，所以放在一起说了\
漏洞点在`sun.rmi.registry.RegistryImpl_Skel#dispatch`动态调一下，跟进两个线程就能到

只截取了bind,其他的差不多

```java
switch (var3) {
    case 0:
        try {
            var11 = var2.getInputStream();
            var7 = (String)var11.readObject();
            var8 = (Remote)var11.readObject();
        } catch (IOException var94) {
            throw new UnmarshalException("error unmarshalling arguments", var94);
        } catch (ClassNotFoundException var95) {
            throw new UnmarshalException("error unmarshalling arguments", var95);
        } finally {
            var2.releaseInputStream();
        }

        var6.bind(var7, var8);

        try {
            var2.getResultStream(true);
            break;
        } catch (IOException var93) {
            throw new MarshalException("error marshalling return", var93);
        }
```

yso有实现 RMIRegistryExploit，是使用的bind，这里的攻击不限制版本（除了JEP290），因为虽说高版本bind需要在同一台机器上，但是高版本bind的校验是在bind函数中，和readObject没有关系

### 客户端攻击服务端

条件比较苛刻，反序列化的点在服务端会对客户端传递的参数进行处理（特定条件进行反序列化）\


```
sun.rmi.server.UnicastServerRef#dispatch
```

```java
try {
    unmarshalCustomCallData(in);
    for (int i = 0; i < types.length; i++) {
        params[i] = unmarshalValue(types[i], in);
    }
}
```

```java
protected static Object unmarshalValue(Class<?> type, ObjectInput in)
    throws IOException, ClassNotFoundException
{
    if (type.isPrimitive()) {
        if (type == int.class) {
            return Integer.valueOf(in.readInt());
        } else if (type == boolean.class) {
            return Boolean.valueOf(in.readBoolean());
        } else if (type == byte.class) {
            return Byte.valueOf(in.readByte());
        } else if (type == char.class) {
            return Character.valueOf(in.readChar());
        } else if (type == short.class) {
            return Short.valueOf(in.readShort());
        } else if (type == long.class) {
            return Long.valueOf(in.readLong());
        } else if (type == float.class) {
            return Float.valueOf(in.readFloat());
        } else if (type == double.class) {
            return Double.valueOf(in.readDouble());
        } else {
            throw new Error("Unrecognized primitive type: " + type);
        }
    } else {
        return in.readObject();
    }
}
```

问题就是攻击者传递的参数一定是Object或者什么类型，而在dispatch的unmarshalValue之前有类型校验，那么就要想办法绕过，大佬提出了以下方法

afanti总结了以下途径：

1. 直接修改rmi底层源码
2. 在运行时，添加调试器hook客户端，然后替换
3. 在客户端使用Javassist工具更改字节码
4. 使用代理，来替换已经序列化的对象中的参数信息

```javascript
    op = in.readLong();
...


Method method = hashToMethod_Map.get(op);
if (method == null) {
                throw new UnmarshalException("unrecognized method hash: " +
                    "method not supported by remote object");
            }
```

而unmarshalValue内的部分只要不是基本类型就可以

（这一块也可以使用在注册中心的攻击，只要再改一下前边的num,让逻辑进不去oldDisptach）

在这个师傅这看到的

{% embed url="https://pwnull.github.io/2022/Exploring-JAVA-RMI's-offensive-and-defensive-history/#3-3-%E7%BB%95%E8%BF%872%EF%BC%9A%E7%BB%95%E8%BF%87%E6%9C%AC%E5%9C%B0%E5%9C%B0%E5%9D%80%E9%99%90%E5%88%B6%EF%BC%88CVE-2019-2684%EF%BC%89" %}

工具

{% embed url="https://github.com/Afant1/RemoteObjectInvocationHandler" %}

### 攻击DGC

实际上是攻击服务端，因为数据包是在和服务端通信过程中的

sun.rmi.transport.DGCImpl\_Skel#dispatch

截取了一部分，因为触发点一样，case1是dirty

```java
switch (var3) {
    case 0:
        VMID var39;
        boolean var40;
        try {
            ObjectInput var14 = var2.getInputStream();
            var7 = (ObjID[])var14.readObject();
            var8 = var14.readLong();
            var39 = (VMID)var14.readObject();
            var40 = var14.readBoolean();
        } catch (IOException var36) {
            throw new UnmarshalException("error unmarshalling arguments", var36);
        } catch (ClassNotFoundException var37) {
            throw new UnmarshalException("error unmarshalling arguments", var37);
        } finally {
            var2.releaseInputStream();
        }

        var6.clean(var7, var8, var39, var40);

        try {
            var2.getResultStream(true);
            break;
        } catch (IOException var35) {
            throw new MarshalException("error marshalling return", var35);
        }
```

ysoserial的exploit\JRMPClient就是攻击的这个位置



## JDK=8u121 JEP290

著名的JEP290

在反序列化时，递归的对被反序列化的名字进行检查，黑名单策略







