# RMI攻击可能利用到的点总结

首先是一些工具的收集整理，后面会一点点的去调试学习

工具主要是从别的博客找到的，会在最后贴出博客地址

首先是对RMI服务的探测，和RMI上开启的服务bind了什么对象，以及这个对象有哪些方法和其接受的参数

BaRMIe



## 客户端攻击注册端

很鸡肋，一般需要客户端攻击服务端，我们需要自动服务端开启的RMI的类名，方法名，参数类型，且参数类型不能为基本类

基本上是参数在某些情况下会在服务端进行反序列化，条件就是不能为基本数据类型

一般来说强制要求为Object才会进行readObject，因为会进行methodHash校验，但是可以通过代理等方式修改流量中的这个值，从而达到绕过。但是下一个问题是，反序列化函数在进行反序列化之前判断了真实的（即代码定义中的参数）参数是不是基本类型，所以参数只要不是基本类型，就可以进行反序列化

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

