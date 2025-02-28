---
description: ObjectOutputStream 和 ObjectInputStream 解读
icon: '0'
---

# java 原生readObject解读

&#x20;有点多，自己动调了一下 等等我

## ObjectInputStream.readObject

假设所有的反序列化来自于(从ysoserial找的)

```
public class UnSerial implements Callable<Object> {
    private final byte[] bytes;

    public UnSerial(byte[] bytes) { this.bytes = bytes; }

    public Object call() throws Exception {
        return deserialize(bytes);
    }

    public static Object deserialize(final byte[] serialized) throws IOException, ClassNotFoundException {
        final ByteArrayInputStream in = new ByteArrayInputStream(serialized);
        return deserialize(in);
    }

    public static Object deserialize(final InputStream in) throws ClassNotFoundException, IOException {
        final ObjectInputStream objIn = new ObjectInputStream(in);
        return objIn.readObject();
    }

    public static void main(String[] args) throws ClassNotFoundException, IOException {
        final InputStream in = args.length == 0 ? System.in : new FileInputStream(new File(args[0]));
        Object object = deserialize(in);
    }
}
```

可以看到一切都是到了ObjectInputStream.readObject

```java
public ObjectInputStream(InputStream in) throws IOException {
        verifySubclass();
        bin = new BlockDataInputStream(in);
        handles = new HandleTable(10);
        vlist = new ValidationList();
        enableOverride = false;
        readStreamHeader(); //读取头信息判断数据类型是否正确
        bin.setBlockDataMode(true);
}
    
public final Object readObject()
    throws IOException, ClassNotFoundException
{
    if (enableOverride) { //默认false 不执行
        return readObjectOverride();
    }

    // if nested read, passHandle contains handle of enclosing object
    int outerHandle = passHandle;
    try {
        Object obj = readObject0(false);
        handles.markDependency(outerHandle, passHandle);
        ClassNotFoundException ex = handles.lookupException(passHandle);
        if (ex != null) {
            throw ex;
        }
        if (depth == 0) {
            vlist.doCallbacks();
        }
        return obj;
    } finally {
        passHandle = outerHandle;
        if (closed && depth == 0) {
            clear();
        }
    }
}
```





## ObjectOutputStream.writeObject

虽然是readObject的解读，但是基于中国人来都来了的原则

看都看了，顺便看一看ObjectOutputStream writeObject

所有的序列化假设都是基于这个实现的(从ysoserial找的)

```java
public class Serial implements Callable<byte[]> {
    private final Object object;
    public Serial(Object object) {
        this.object = object;
    }

    public byte[] call() throws Exception {
        return serialize(object);
    }

    public static byte[] serialize(final Object obj) throws IOException {
        final ByteArrayOutputStream out = new ByteArrayOutputStream();
        serialize(obj, out);
        return out.toByteArray();
    }

    public static void serialize(final Object obj, final OutputStream out) throws IOException {
        final ObjectOutputStream objOut = new ObjectOutputStream(out);
        objOut.writeObject(obj);
    }

}
```

那看流程所有的序列化都会到ObjectOutputStream.writeObject（关于ObjectOutputStream的构造函数就不说了）

<pre class="language-java"><code class="lang-java"><strong>public final void writeObject(Object obj) throws IOException {
</strong>    if (enableOverride) {
        writeObjectOverride(obj);
        return;
    }
    try {
        writeObject0(obj, false);
    } catch (IOException ex) {
        if (depth == 0) {
            writeFatalException(ex);
        }
        throw ex;
    }
}
</code></pre>

enableOverride在构造函数里是被赋值为false的，所以继续看writeObject0

很长很长，精简着说（省略输入输出部分，格式化部分，着重于数据操作部分）
