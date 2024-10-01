> 笔记来源：[尚硅谷JVM全套教程，百万播放，全网巅峰（宋红康详解java虚拟机）](https://www.bilibili.com/video/BV1PJ411n7xZ "尚硅谷JVM全套教程，百万播放，全网巅峰（宋红康详解java虚拟机）")
>
> 同步更新：https://gitee.com/vectorx/NOTE_JVM
>
> https://codechina.csdn.net/qq_35925558/NOTE_JVM
>
> https://github.com/uxiahnan/NOTE_JVM

[toc]

# 1. 概述

类加载器是 JVM 执行类加载机制的前提。

**ClassLoader的作用：**

ClassLoader是Java的核心组件，所有的Class都是由ClassLoader进行加载的，`ClassLoader负责通过各种方式将Class信息的二进制数据流读入JVM内部，转换为一个与目标类对应的java.lang.Class对象实例。然后交给Java虚拟机进行链接、初始化等操作。因此，ClassLoader在整个装载阶段，只能影响到类的加载，而无法通过ClassLoader去改变类的链接和初始化行为。`至于它是否可以运行，则由Execution Engine决定。

![image-20210501102535142](https://img-blog.csdnimg.cn/img_convert/fb51cabb2218d857a809a59918c5beec.png)

类加载器最早出现在Java1.Q版本中，那个时候只是单纯地为了满足Java Applet应用而被研发出来。但如今类加载器却在OSGi、字节码加解密领域大放异彩.这主要归功于Java虚拟机的设计者们当初在设计类加载器的时候，并没有考虑将它绑定在JVM内部，这样做的好处就是能够更加灵活和动态地执行类加载操作。

## 1.1. 大厂面试题

> <mark>蚂蚁金服：</mark>
>
> 深入分析ClassLoader，双亲委派机制
>
> 类加载器的双亲委派模型是什么？一面：双亲委派机制及使用原因
>
> 
>
> <mark>百度：</mark>
>
> 都有哪些类加载器，这些类加载器都加载哪些文件？
>
> 手写一个类加载器Demo
>
> Class的forName（“java.lang.String”）和Class的getClassLoader（）的Loadclass（“java.lang.String”）有什么区别？
>
> 
>
> <mark>腾讯：</mark>
>
> 什么是双亲委派模型？
>
> 类加载器有哪些？
>
> 
>
> <mark>小米：</mark>
>
> 双亲委派模型介绍一下
>
> 
>
> <mark>滴滴：</mark>
>
> 简单说说你了解的类加载器一面：讲一下双亲委派模型，以及其优点
>
> 
>
> <mark>字节跳动：</mark>
>
> 什么是类加载器，类加载器有哪些？
>
> 
>
> <mark>京东：</mark>
>
> 类加载器的双亲委派模型是什么？
>
> 双亲委派机制可以打破吗？为什么

## 1.2. 类加载器的分类

**类的加载分类：显式加载 vs 隐式加载**

class文件的显式加载与隐式加载的方式是指JVM加载class文件到内存的方式。

- 显式加载指的是在代码中通过调用ClassLoader加载class对象，如直接使用`Class.forName(name)` `this.getClass().getClassLoader().loadClass()`加载class对象。
- 隐式加载则是不直接在代码中调用ClassLoader的方法加载class对象，而是通过虚拟机自动加载到内存中，如在加载某个类的class文件时，该类的class文件中引用了另外一个类的对象，此时额外引用的类将通过JVM自动加载到内存中。

在日常开发以上两种方式一般会混合使用。

```java
//隐式加载
User user=new User();
//显式加载，并初始化
Class clazz=Class.forName("com.test.java.User");
//显式加载，但不初始化
ClassLoader.getSystemClassLoader().loadClass("com.test.java.Parent"); 
```

## 1.3. 类加载器的必要性

一般情况下，Java开发人员并不需要在程序中显式地使用类加载器，但是了解类加载器的加载机制却显得至关重要。从以下几个方面说：

- 避免在开发中遇到java.lang.ClassNotFoundException异常或java.lang.NoClassDefFoundError异常时，手足无措。只有了解类加载器的 加载机制才能够在出现异常的时候快速地根据错误异常日志定位问题和解决问题
- 需要支持类的动态加载或需要对编译后的字节码文件进行加解密操作时，就需要与类加载器打交道了。
- 开发人员可以在程序中编写自定义类加载器来重新定义类的加载规则，以便实现一些自定义的处理逻辑。

## 1.4. 命名空间

**何为类的唯一性？**

$\color{red}{对于任意一个类，都需要由加载它的类加载器和这个类本身一同确认其在Java虚拟机中的唯一性。}$每一个类加载器，都拥有一个独立的类名称空间：$\color{red}{比较两个类是否相等，只有在这两个类是由同一个类加载器加载的前提下才有意义。}$否则，即使这两个类源自同一个Class文件，被同一个虚拟机加载，只要加载他们的类加载器不同，那这两个类就必定不相等。

**命名空间**

- `每个类加载器都有自己的命名空间`，命名空间由该加载器及所有的父加载器所加载的类组成

- 在同一命名空间中，不会出现类的完整名字（包括类的包名）相同的两个类

- 在不同的命名空间中，有可能会出现类的完整名字（包括类的包名）相同的两个类

在大型应用中，我们往往借助这一特性，来运行同一个类的不同版本。

<img src="/Users/yannlau/Documents/JavaSet/Java韩顺平/高阶专题/JVM专题/NOTE_JVM_宋红康/JVM中篇：字节码与类的加载篇/04-再谈类的加载器/assets/QQ_1727398725739.png" alt="QQ_1727398725739" style="zoom:50%;" />

```java
package com.classloader;

import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.InvocationTargetException;

/**
 * @author YannLau
 * @version 1.0
 * @program untitled
 * @ClassName UserClassLoader
 * @ClassPath com.classloader.UserClassLoader
 * @create 2024-09-27 08:35
 * @description
 */
public class UserClassLoader extends ClassLoader {
    private String rootDir;

    public UserClassLoader(String rootDir) {
        this.rootDir = rootDir;
    }

    /**
     * @param name
     * @throws
     * @Author YannLau
     * @Description 编写findClass的逻辑
     * @Date 2024/9/27
     * @Time 8:36 AM
     */
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        //获取类的class文件的字节数组
        byte[] classData = getClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException(name);
        } else {
            //直接生成class对象
            return defineClass(name, classData, 0, classData.length);
        }
    }

    /**
     * @param null
     * @throws
     * @Author YannLau
     * @Description 编写获取class文件并转换为字节码流的逻辑
     * @Date 2024/9/27
     * @Time 8:40 AM
     */
    private byte[] getClassData(String className) {
        //读取类文件的字节
        String path = classNameToPath(className);
        try (InputStream ins = new FileInputStream(path)) {
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            byte[] buffer = new byte[1024];
            int len = 0;
            // 读取类文件的字节码
            while ((len = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, len);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    private String classNameToPath(String className) {
        return rootDir + "/" + className.replace('.', '/') + ".class";
    }

    public static void main(String[] args) {
        UserClassLoader userClassLoader1 = new UserClassLoader("/Users/yannlau/Documents/ProjectRepository/IdeaProjects/untitled/src");
        UserClassLoader userClassLoader2 = new UserClassLoader("/Users/yannlau/Documents/ProjectRepository/IdeaProjects/untitled/src");
        try {
            Class<?> aClass1 = userClassLoader1.findClass("com.classloader.User"); //UserClassLoader的实例1加载的User类
            Class<?> aClass2 = userClassLoader2.findClass("com.classloader.User"); //UserClassLoader的实例2加载的User类
            Class<?> aClass3 = User.class;  //系统类加载器(应用程序类加载器加载的User类)
            User user = new User();
            Object user1 = aClass1.getConstructor().newInstance();
            System.out.println(user);
            System.out.println(user1);
            System.out.println(user.getClass() == user1.getClass());
            System.out.println(aClass1.hashCode());
            System.out.println(aClass2.hashCode());
            System.out.println(aClass3.hashCode());
        } catch (ClassNotFoundException | InvocationTargetException | InstantiationException | IllegalAccessException |
                 NoSuchMethodException e) {
            throw new RuntimeException(e);
        }
    }
}
//输出结果
com.classloader.User@63961c42
com.classloader.User@65b54208
false
1450495309
468121027
140435067
```

> 02-复习：类的加载器分类
> JVM支持两种类型的类加载器，分别为引导类加载器（Bootstrap ClassLoader）和自定义类加载器（User-Defined ClassLoader)
>
> 从概念上来讲，自定义类加载器一般指的是程序中由开发人员自定义的一类类加载器，但是Java虚拟机规范却没有这么定义，而是将所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器。无论类加载器的类型如何划分，在程序中我们最常见的类加载器结构主要是如下情况：
>
> 下图中除了Bootstrap加载器使用C++实现的，其他都是用Java语言实现的，所以都是自定义类加载器。
>
> <img src="/Users/yannlau/Documents/JavaSet/Java韩顺平/高阶专题/JVM专题/NOTE_JVM_宋红康/JVM中篇：字节码与类的加载篇/04-再谈类的加载器/assets/QQ_1727398951695.png" alt="QQ_1727398951695" style="zoom:50%;" />
>
> • 除了顶层的启动类加载器外，其余的类加载器都应当有自己的`“父类”`加载器。
> • ==不同类加载器看似是继承（Inheritance）关系，实际上是包含关系。在下层加载器中，包含着上层加载器的引用==
>
> 就比如系统类加载器的父加载器是扩展类加载器，但是在语法上的extends两者解释URLClassLoader,间接继承了ClassLoader。
>
> ```java
> class ClassLoader{
>   ClassLoader parent;//父类加载器
>   public ClassLoader(ClassLoader parent){
>     this.parent = parent;
>   }
> }
> 
> class ParentClassLoader extends ClassLoader{
> 	public ParentClassLoader(ClassLoader parent){
>     super(parent);
>   }
>   
> }
> class ChildClassLoader extends ClassLoader{
>   public ChildClassLoader(ClassLoader parent){  //parent = new ParentClasLoader(); 传给这里的构造器，形成聚合关系
>     super(parent);
>   }
> }
> ```
>
> 具体区别看上篇笔记

## 1.5. 类加载机制的基本特征

通常类加载机制有三个基本特征：

==双亲委派模型==。但不是所有类加载都遵守这个模型，有的时候，启动类加载器所加载的类型，是可能要加载用户代码的，比如JDK内部的ServiceProvider/ServiceLoader机制，用户可以在标准API框架上，提供自己的实现，JDK也需要提供些默认的参考实现。例如，Java中JNDI、JDBC、文件系统、Cipher等很多方面，都是利用的这种机制，这种情况就不会用双亲委派模型去加载，而是利用所谓的`上下文加载器`。

<mark>可见性</mark>，`子类加载器可以访问父加载器加载的类型，但是反过来是不允许的。`不然，因为缺少必要的隔离，我们就没有办法利用类加载器去实现容器的逻辑。

<mark>单一性</mark>，由于父加载器的类型对于子加载器是可见的，所以父加载器中加载过的类型，就不会在子加载器中重复加载。但是注意，类加载器“邻居”间，同一类型仍然可以被加载多次，因为互相并不可见。

##  1.6. 类加载器之间的关系

Launcher类核心代码

```java
Launcher.ExtClassLoader var1;
try {
    var1 = Launcher.ExtClassLoader.getExtClassLoader();
} catch (IOException var10) {
    throw new InternalError("Could not create extension class loader", var10);
}

try {
    this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
} catch (IOException var9) {
    throw new InternalError("Could not create application class loader", var9);
}

Thread.currentThread().setContextClassLoader(this.loader);
```

- **ExtClassLoader的Parent类是null**

- **AppClassLoader的Parent类是ExtClassLoader**

- **当前线程的ClassLoader是AppClassLoader**

$\color{red}{注意，这里的Parent类并不是Java语言意义上的继承关系，而是一种包含关系}$

<hr/>

# 2. 类的加载器分类

JVM支持两种类型的类加载器，分别为引导类加载器（Bootstrap ClassLoader）和自定义类加载器（User-Defined ClassLoader）。

从概念上来讲，自定义类加载器一般指的是程序中由开发人员自定义的一类类加载器，但是Java虚拟机规范却没有这么定义，而是将所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器。无论类加载器的类型如何划分，在程序中我们最常见的类加载器结构主要是如下情况：

![image-20210501164413665](https://img-blog.csdnimg.cn/img_convert/0c43fb4a7da20038c8f56b42a1ddf802.png)

- 除了顶层的启动类加载器外，其余的类加载器都应当有自己的“父类”加戟器。
- 不同类加载器看似是继承（Inheritance）关系，实际上是包含关系。在下层加载器中，包含着上层加载器的引用。

父类加载器和子类加载器的关系：

```java
class ClassLoader{
    ClassLoader parent;//父类加载器
        public ClassLoader(ClassLoader parent){
        this.parent = parent;
    }
}
class ParentClassLoader extends ClassLoader{
    public ParentClassLoader(ClassLoader parent){
        super(parent);
    }
}
class ChildClassLoader extends ClassLoader{
    public ChildClassLoader(ClassLoader parent){ //parent = new ParentClassLoader();
        super(parent);
    }
}
```

正是由于子类加载器中包含着父类加载器的引用，所以可以通过子类加载器的方法获取对应的父类加载器

**注意：**

启动类加载器通过C/C++语言编写，而自定义类加载器都是由Java语言编写的，虽然扩展类加载器和应用程序类加载器是被jdk开发人员使用java语言来编写的，但是也是由java语言编写的，所以也被称为自定义类加载器

## 2.1. 引导类加载器

<mark>启动类加载器（引导类加载器，Bootstrap ClassLoader）</mark>

- 这个类加载使用C/C++语言实现的，嵌套在JVM内部。

- 它用来加载Java的核心库（JAVAHOME/jre/lib/rt.jar或sun.boot.class.path路径下的内容）。用于提供JVM自身需要的类。

- 并不继承自java.lang.ClassLoader，没有父加载器。

- 出于安全考虑，Bootstrap启动类加载器只加载包名为java、javax、sun等开头的类

- 加载扩展类和应用程序类加载器，并指定为他们的父类加载器。

  ![image-20210501170011811](https://img-blog.csdnimg.cn/img_convert/43431a1abff0e5f2bd7bfd20fe91e5f7.png)
  ![image-20210501170038212](https://img-blog.csdnimg.cn/img_convert/5cd73234cd993408846ec7b4c2cb7238.png)
  使用-XX:+TraceClassLoading参数得到。

启动类加载器使用C++编写的？Yes！

  - C/C++：指针函数&函数指针、C++支持多继承、更加高效
  - Java：由C++演变而来，（C++）–版，单继承

```java
System.out.println("＊＊＊＊＊＊＊＊＊＊启动类加载器＊＊＊＊＊＊＊＊＊＊");
// 获取BootstrapclassLoader能够加载的api的路径
URL[] urLs = sun.misc.Launcher.getBootstrapcLassPath().getURLs();
for (URL element : urLs) {
    System.out.println(element.toExternalForm());
}
// 从上面的路径中随意选择一个类，来看看他的类加载器是什么：引导类加载器
ClassLoader classLoader = java.security.Provider.class.getClassLoader();
System.out.println(classLoader);
```

**执行结果：**
![image-20210501170425889](https://img-blog.csdnimg.cn/img_convert/c72286acf05c7f86d1ea24f74e0c2a1e.png)

## 2.2. 扩展类加载器

<mark>扩展类加载器（Extension ClassLoader）</mark>

- Java语言编写，由sun.misc.Launcher$ExtClassLoader实现。

- 继承于ClassLoader类

- 父类加载器为启动类加载器

- 从java.ext.dirs系统属性所指定的目录中加载类库，或从JDK的安装目录的jre/lib/ext子目录下加载类库。如果用户创建的JAR放在此目录下，也会自动由扩展类加载器加载。

  ![在这里插入图片描述](https://img-blog.csdnimg.cn/img_convert/ba54af96e744eb99a9248d13e55a7e3c.png)

```java
System.out.println("＊＊＊＊＊＊＊＊＊＊＊扩展类加载器＊＊＊＊＊＊＊＊＊＊＊");
String extDirs =System.getProperty("java.ext.dirs");
for (String path :extDirs.split( regex:";")){
    System.out.println(path);
}

// 从上面的路径中随意选择一个类，来看看他的类加载器是什么：扩展类加载器
lassLoader classLoader1 = sun.security.ec.CurveDB.class.getClassLoader();
System.out.print1n(classLoader1); //sun.misc. Launcher$ExtCLassLoader@1540e19d
```

**执行结果：**

  ![img](https://img-blog.csdnimg.cn/img_convert/b21313ee65acb4f01f1d03b88529e9f5.png)

## 2.3. 系统类加载器

<mark>应用程序类加载器（系统类加载器，AppClassLoader）</mark>

- java语言编写，由sun.misc.Launcher$AppClassLoader实现
- 继承于ClassLoader类
- 父类加载器为扩展类加载器
- 它负责加载环境变量classpath或系统属性java.class.path 指定路径下的类库
- $\color{red}{应用程序中的类加载器默认是系统类加载器。}$
- 它是用户自定义类加载器的默认父加载器
- 通过ClassLoader的getSystemClassLoader()方法可以获取到该类加载器

![image-20210501171206453](https://img-blog.csdnimg.cn/img_convert/19dd9595afeb6543593ef14161fc1bb8.png)

## 2.4. 用户自定义类加载器

<mark>用户自定义类加载器</mark>

- 在Java的日常应用程序开发中，类的加载几乎是由上述3种类加载器相互配合执行的。在必要时，我们还可以自定义类加载器，来定制类的加载方式。
- 体现Java语言强大生命力和巨大魅力的关键因素之一便是，Java开发者可以自定义类加载器来实现类库的动态加载，加载源可以是本地的JAR包，也可以是网络上的远程资源。
- $\color{red}{通过类加载器可以实现非常绝妙的插件机制}$，这方面的实际应用案例举不胜举。例如，著名的OSGI组件框架，再如Eclipse的插件机制。类加载器为应用程序提供了一种动态增加新功能的机制，这种机制无须重新打包发布应用程序就能实现。
- 同时，$\color{red}{自定义加载器能够实现应用隔离}$，例如Tomcat，Spring等中间件和组件框架都在内部实现了自定义的加载器，并通过自定义加载器隔离不同的组件模块。这种机制比C/C++程序要好太多，想不修改C/C++程序就能为其新增功能，几乎是不可能的，仅仅一个兼容性便能阻挡住所有美好的设想。
- 自定义类加载器通常需要继承于ClassLoader。

<hr/>

# 3. 测试不同的类的加载器

每个Class对象都会包含一个定义它的ClassLoader的一个引用。
**获取ClassLoader的途径**

```java
// 获得当前类的ClassLoader
clazz.getClassLoader()
// 获得当前线程上下文的ClassLoader
Thread.currentThread().getContextClassLoader()
// 获得系统的ClassLoader
ClassLoader.getSystemClassLoader()
```

**说明：**

- 站在程序的角度看，引导类加载器与另外两种类加载器（系统类加载器和扩展类加载器）并不是同一个层次意义上的加
  载器，引导类加载器是使用C++语言编写而成的，而另外两种类加载器则是使用Java语言编写而成的。由于引导类加载
  器压根儿就不是一个Java类，因此在Java程序中只能打印出空值。
- 数组类的Class对象，不是由类加载器去创建的，而是在Java运行期JVM根据需要自动创建的。对于数组类的类加载器
  来说，是通过Class.getClassLoader()返回的，`数组类的加载器与数组当中元素类型的类加载器是一样的`；如果数组当中的元素类型
  是基本数据类型，数组类是没有类加载器的。

```java
// 运行结果：null
String[] strArr = new String[6];
System.out.println(strArr.getClass().getClassLoader());

// 运行结果：sun．misc．Launcher＄AppCLassLoader＠18b4aac2
ClassLoaderTest[] test=new ClassLoaderTest[1];
System.out.println(test.getClass().getClassLoader());

// 运行结果：null
int[]ints =new int[2];
System.out.println(ints.getClass().getClassLoader());
```

**代码：**

```java
public class ClassLoaderTest1{
    public static void main(String[] args) {
        //获取系统该类加载器
        ClassLoader systemClassLoader=ClassLoader.getSystemCLassLoader();
        System.out.print1n(systemClassLoader);//sun.misc.Launcher$AppCLassLoader@18b4aac2
        //获取扩展类加载器
        ClassLoader extClassLoader =systemClassLoader.getParent();
        System.out.println(extClassLoader);//sun.misc. Launcher$ExtCLassLoader@1540e19d
        //试图获取引导类加载器：失败
        ClassLoader bootstrapClassLoader =extClassLoader.getParent();
        System.out.print1n(bootstrapClassLoader);//null

        //##################################
        try{
            ClassLoader classLoader =Class.forName("java.lang.String").getClassLoader();
            System.out.println(classLoader);
            //自定义的类默认使用系统类加载器
            ClassLoader classLoader1=Class.forName("com.atguigu.java.ClassLoaderTest1").getClassLoader();
            System.out.println(classLoader1);
            
            //关于数组类型的加载：使用的类的加载器与数组元素的类的加载器相同
            String[] arrstr = new String[10];
            System.out.println(arrstr.getClass().getClassLoader());//null：表示使用的是引导类加载器
                
            ClassLoaderTest1[] arr1 =new ClassLoaderTest1[10];
            System.out.println(arr1.getClass().getClassLoader());//sun.misc. Launcher$AppcLassLoader@18b4aac2
            
            int[] arr2 = new int[10];
            System.out.println(arr2.getClass().getClassLoader());//null:
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

<hr/>

# 4. ClassLoader源码解析

**ClassLoader与现有类的关系：**

<img src="/Users/yannlau/Documents/JavaSet/Java韩顺平/高阶专题/JVM专题/NOTE_JVM_宋红康/JVM中篇：字节码与类的加载篇/04-再谈类的加载器/assets/QQ_1727404252800.png" alt="QQ_1727404252800" style="zoom:50%;" />

除了以上虚拟机自带的加载器外，用户还可以定制自己的类加载器。Java提供了抽象类java.lang.ClassLoader（但是不含抽象方法），所有用户自定义的类加载器都应该继承ClassLoader类。

## 4.1. ClassLoader的主要方法

抽象类ClassLoader的主要方法：（内部没有抽象方法）

```java
public final ClassLoader getParent()
```

返回该类加载器的超类加载器

```java
public Class<?> loadClass(String name) throws ClassNotFoundException
```

<img src="/Users/yannlau/Documents/JavaSet/Java韩顺平/高阶专题/JVM专题/NOTE_JVM_宋红康/JVM中篇：字节码与类的加载篇/04-再谈类的加载器/assets/QQ_1727406418018.png" alt="QQ_1727406418018" style="zoom:50%;" />

<img src="/Users/yannlau/Documents/JavaSet/Java韩顺平/高阶专题/JVM专题/NOTE_JVM_宋红康/JVM中篇：字节码与类的加载篇/04-再谈类的加载器/assets/QQ_1727406457238.png" alt="QQ_1727406457238" style="zoom:50%;" />

加载名称为name的类，返回结果为java.lang.Class类的实例。如果找不到类，则返回 ClassNotFoundException异常。该方法中的逻辑就是双亲委派模式的实现。

```java
protected Class<?> findClass(String name) throws ClassNotFoundException
```

查找二进制名称为name的类，返回结果为java.lang.Class类的实例。这是一个受保护的方法，JVM鼓励我们重写此方法，需要自定义加载器遵循双亲委托机制，该方法会在检查完父类加载器之后被loadClass()方法调用。

- 在JDK1.2之前，在自定义类加载时，总会去继承ClassLoader类并重写loadClass方法，从而实现自定义的类加载类。但是在JDK1.2之后已不再建议用户去覆盖loadClass()方法，而是建议把自定义的类加载逻辑写在findClass()方法中，从前面的分析可知，findClass()方法是在loadClass()方法中被调用的，当loadClass()方法中父加载器加载失败后，则会调用自己的findClass()方法来完成类加载，这样就可以保证自定义的类加载器也符合双亲委托模式。

- 需要注意的是ClassLoader类中并没有实现findClass()方法的具体代码逻辑，取而代之的是抛出ClassNotFoundException异常，同时应该知道的是findClass方法通常是和defineClass方法一起使用的。$\color{red}{一般情况下，在自定义类加载器时，会直接覆盖ClassLoader的findClass()方法并编写加载规则，取得要加载类的字节码后转换成流，然后调用defineClass()方法生成类的Class对象。}$

```java
protected final Class<?> defineClass(String name, byte[] b,int off,int len)
```

根据给定的字节数组b转换为Class的实例，off和len参数表示实际Class信息在byte数组中的位置和长度，其中byte数组b是ClassLoader从外部获取的。这是受保护的方法，只有在自定义ClassLoader子类中可以使用。

- defineClass()方法是用来将byte字节流解析成JVM能够识别的Class对象（ClassLoader中已实现该方法逻辑），通过这个方法不仅能够通过class文件实例化class对象，也可以通过其他方式实例化class对象，如通过网络接收一个类的字节码，然后转换为byte字节流创建对应的Class对象。

- $\color{red}{defineClass()方法通常与findClass()方法一起使用，一般情况下，在自定义类加载器时，会直接覆盖ClassLoader的findClass()方法并编写加载规则，取得要加载类的字节码后转换成流，然后调用defineClass()方法生成类的Class对象}$

**简单举例：**

```java
protected Class<?> findClass(String name) throws ClassNotFoundException {
    // 获取类的字节数组
    byte[] classData =getClassData(name);
    if (classData == null) {
        throw new ClassNotFoundException();
    } else{
        //使用defineClass生成class对象
        return defineClass(name,classData,θ,classData.length);
    }
}
```

```java
protected final void resolveClass(Class<?> c)
```

链接指定的一个Java类。使用该方法可以使用类的Class对象创建完成的同时也被解析。前面我们说链接阶段主要是对字节码进行验证，为类变量分配内存并设置初始值同时将字节码文件中的符号引用转换为直接引用。

```java
protected final Class<?> findLoadedClass(String name)
```

查找名称为name的已经被加载过的类，返回结果为java.lang.Class类的实例。这个方法是final方法，无法被修改。

```java
private final ClassLoader parent;
```

它也是一个ClassLoader的实例，这个字段所表示的ClassLoader也称为这个ClassLoader的双亲。在类加载的过程中，ClassLoader可能会将某些请求交予自己的双亲处理。

## 4.2. SecureClassLoader与URLClassLoader

接着SecureClassLoader扩展了ClassLoader，新增了几个与使用相关的代码源（对代码源的位置及其证书的验证）和权限定义类验证（主要指对class源码的访问权限）的方法，一般我们不会直接跟这个类打交道，更多是与它的子类URLClassLoader有所关联。

前面说过，ClassLoader是一个抽象类，很多方法是空的没有实现，比如findClass()、findResource()等。而URLClassLoader这个实现类为这些方法提供了具体的实现。并新增了URLClassPath类协助取得Class字节码流等功能。$\color{red}{在编写自定义类加载器时，如果没有太过于复杂的需求，可以直接继承URLClassLoader类}$，这样就可以避免自己去编写findClass()方法及其获取字节码流的方式，使自定义类加载器编写更加简洁。

![image-20210501174730756](https://img-blog.csdnimg.cn/img_convert/53f634bada23b3400a303bfcc6e11a31.png)

## 4.3. ExtClassLoader与AppClassLoader

了解完URLClassLoader后接着看看剩余的两个类加载器，即拓展类加载器ExtClassLoader和系统类加载器AppClassLoader，这两个类都继承自URLClassLoader，是sun.misc.Launcher的静态内部类。

sun.misc.Launcher主要被系统用于启动主应用程序，ExtClassLoader和AppClassLoader都是由sun.misc.Launcher创建的，其类主要类结构如下：

![img](https://img-blog.csdnimg.cn/img_convert/a77b6bca10308e6b9be35b1b2dfc84bc.png)

我们发现ExtClassLoader并没有重写loadClass()方法，这足矣说明其遵循双亲委派模式，而AppClassLoader重载了loadClass()方法，但最终调用的还是父类loadClass()方法，因此依然遵守双亲委派模式。

## 4.4. Class.forName()与ClassLoader.loadClass()

**Class.forName()**

- Class.forName()：是一个静态方法，最常用的是Class.forName(String className);

- 根据传入的类的全限定名返回一个Class对象。`该方法在将Class文件加载到内存的同时，会执行类的初始化。`

  ```java
  Class.forName("com.atguigu.java.Helloworld");
  ```

**ClassLoader.loadClass()**

- ClassLoader.loadClass()：这是一个实例方法，需要一个ClassLoader对象来调用该方法。

- `该方法将Class文件加载到内存时，并不会执行类的初始化，直到这个类第一次使用时才进行初始化。`该方法因为需要得到一个ClassLoader对象，所以可以根据需要指定使用哪个类加载器。

  ```java
  Classloader cl = ......; cl.loadClass("com.atguigu.java.Helloworld");
  ```

  <hr/>

# 5. 双亲委派模型

## 5.1. 定义与本质

类加载器用来把类加载到Java虚拟机中。从JDK1.2版本开始，类的加载过程采用双亲委派机制，这种机制能更好地保证Java平台的安全。

**定义**

如果一个类加载器在接到加载类的请求时，它首先不会自己尝试去加载这个类，而是把这个请求任务委托给父类加载器去完成，依次递归，如果父类加载器可以完成类加载任务，就成功返回。只有父类加载器无法完成此加载任务时，才自己去加载。

**本质**

规定了类加载的顺序是：引导类加载器先加载，若加载不到，由扩展类加载器加载，若还加载不到，才会由系统类加载器或自定义的类加载器进行加载。

![image-20210501175529542](https://img-blog.csdnimg.cn/img_convert/41cbfd9ba1da676a7f05dae0dac3abae.png)

![img](https://img-blog.csdnimg.cn/img_convert/c1526b63e95852b48551b4f136ebbd6f.png)

## 5.2. 优势与劣势

**双亲委派机制优势**

- 避免类的重复加载，确保一个类的全局唯一性

  $\color{red}{Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关可以避免类的重复加载，当父亲已经加载了该类时，就没有必要子ClassLoader再加载一次。}$

- 保护程序安全，防止核心API被随意篡改

**代码支持**

双亲委派机制在java.lang.ClassLoader.loadClass(String，boolean)接口中体现。该接口的逻辑如下：

（1）先在当前加载器的缓存中查找有无目标类，如果有，直接返回。

（2）判断当前加载器的父加载器是否为空，如果不为空，则调用parent.loadClass(name，false)接口进行加载。

（3）反之，如果当前加载器的父类加载器为空，则调用findBootstrapClassorNull(name)接口，让引导类加载器进行加载。

（4）如果通过以上3条路径都没能成功加载，则调用findClass(name)接口进行加载。该接口最终会调用java.lang.ClassLoader接口的defineClass系列的native接口加载目标Java类。

双亲委派的模型就隐藏在这第2和第3步中。

**举例**

假设当前加载的是java.lang.Object这个类，很显然，该类属于JDK中核心得不能再核心的一个类，因此一定只能由引导类加载器进行加载。当JVM准备加载java.lang.Object时，JVM默认会使用系统类加载器去加载，按照上面4步加载的逻辑，在第1步从系统类的缓存中肯定查找不到该类，于是进入第2步。由于从系统类加载器的父加载器是扩展类加载器，于是扩展类加载器继续从第1步开始重复。由于扩展类加载器的缓存中也一定查找不到该类，因此进入第2步。扩展类的父加载器是null，因此系统调用findBootstrapClassOrNull（String），最终通过引导类加载器进行加载。

**思考**

如果在自定义的类加载器中重写java.lang.ClassLoader.loadClass(String)或java.lang.ClassLoader.loadclass(String，boolean)方法，抹去其中的双亲委派机制，仅保留上面这4步中的第l步与第4步，那么是不是就能够加载核心类库了呢？

这也不行！因为JDK还为核心类库提供了一层保护机制。不管是自定义的类加载器，还是系统类加载器抑或扩展类加载器，最终都必须调用 java.lang.ClassLoader.defineclass(String，byte[]，int，int，ProtectionDomain)方法，而该方法会执行preDefineClass()接口，该接口中提供了对JDK核心类库的保护。

**弊端**

检查类是否加载的委托过程是单向的，这个方式虽然从结构上说比较清晰，使各个ClassLoader的职责非常明确，但是同时会带来一个问题，即`顶层的ClassLoader无法访问底层的ClassLoader所加载的类`。

通常情况下，启动类加载器中的类为系统核心类，包括一些重要的系统接口，而在应用类加载器中，为应用类。按照这种模式，`应用类访问系统类自然是没有问题，但是系统类访问应用类就会出现问题`。比如在系统类中提供了一个接口，该接口需要在应用类中得以实现，该接口还绑定一个工厂方法，用于创建该接口的实例，而接口和工厂方法都在启动类加载器中。这时，就会出现该工厂方法无法创建由应用类加载器加载的应用实例的问题。

**结论**

$\color{red}{由于Java虚拟机规范并没有明确要求类加载器的加载机制一定要使用双亲委派模型，只是建议采用这种方式而已。}$比如在Tomcat中，类加载器所采用的加载机制就和传统的双亲委派模型有一定区别，当缺省的类加载器接收到一个类的加载任务时，首先会由它自行加载，当它加载失败时，才会将类的加载任务委派给它的超类加载器去执行，这同时也是Serylet规范推荐的一种做法。

## 5.3. 破坏双亲委派机制

双亲委派模型并不是一个具有强制性约束的模型，而是Java设计者推荐给开发者们的类加载器实现方式。

在Java的世界中大部分的类加载器都遵循这个模型，但也有例外的情况，`直到Java模块化出现为止，双亲委派模型主要出现过3次较大规模“被破坏”的情况`。

**第一次破坏双亲委派机制**

双亲委派模型的第一次“被破坏”其实发生在双亲委派模型出现之前一—即JDK1.2面世以前的“远古”时代。

由于双亲委派模型在JDK 1.2之后才被引入，但是类加载器的概念和抽象类java.lang.ClassLoader则在Java的第一个版本中就已经存在，面对经存在的用户自定义类加载器的代码，Java设计者们引入双亲委派模型时不得不做出一些妥协，$\color{red}{为了兼容这些已有代码，无法再以技术手段避免loadClass()被子类覆盖的可能性}$，只能在JDK1.2之后的java.lang.ClassLoader中添加一个新的protected方法findClass()，并引导用户编写的类加载逻辑时尽可能去重写这个方法，而不是在loadClass()中编写代码。上节我们已经分析过loadClass()方法，双亲委派的具体逻辑就实现在这里面，按照loadClass()方法的逻辑，如果父类加载失败，会自动调用自己的findClass()方法来完成加载，这样既不影响用户按照自己的意愿去加载类，又可以保证新写出来的类加载器是符合双亲委派规则的。

**第二次破坏双亲委派机制：线程上下文类加载器**

双亲委派模型的第二次“被破坏”是由这个模型自身的缺陷导致的，双亲委派很好地解决了各个类加载器协作时基础类型的一致性问题（$\color{red}{越基础的类由越上层的加载器进行加载}$），基础类型之所以被称为“基础”，是因为它们总是作为被用户代码继承、调用的API存在，但程序设计往往没有绝对不变的完美规则，如果有$\color{red}{基础类型又要调用回用户的代码，那该怎么办呢？}$

这并非是不可能出现的事情，一个典型的例子便是JNDI服务，JNDI现在已经是Java的标准服务，它的代码由启动类加载器来完成加载（在JDK 1.3时加入到rt.jar的），肯定属于Java中很基础的类型了。但JNDI存在的目的就是对资源进行查找和集中管理，它需要调用由其他厂商实现并部署在应用程序的ClassPath下的JNDI服务提供者接口（Service Provider Interface，SPI）的代码，现在问题来了，$\color{red}{启动类加载器是绝不可能认识、加载这些代码的，那该怎么办？}$<u>（SPI：在Java平台中，通常把核心类rt.jar中提供外部服务、可由应用层自行实现的接口称为SPI）</u>

为了解决这个困境，Java的设计团队只好引入了一个不太优雅的设计：$\color{red}{线程上下文类加载器（Thread Context ClassLoader）}$。这个类加载器可以通过java.lang.Thread类的setContextClassLoader()方法进行设置，如果创建线程时还未设置，它将会从父线程中继承一个，如果在应用程序的全局范围内都没有设置过的话，那这个类加载器默认就是应用程序类加载器。

有了线程上下文类加载器，程序就可以做一些“舞弊”的事情了。JNDI服务使用这个线程上下文类加载器去加载所需的SPI服务代码，$\color{red}{这是一种父类加载器去请求子类加载器完成类加载的行为，这种行为实际上是打通了双亲委派模型的层次结构来逆向使用类加载器，已经违背了双亲委派模型的一般性原则}$，但也是无可奈何的事情。 ，例如JNDI、JDBC、JCE、JAXB和JBI等。不过，当SPI的服务提供者多于一个的时候，代码就只能根据具体提供者的类型来硬编码判断，为了消除这种极不优雅的实现方式，在JDK6时，JDK提供了java.util.ServiceLoader类，以META-INF/services中的配置信息，辅以责任链模式，这才算是给SPI的加载提供了一种相对合理的解决方案。

![img](https://img-blog.csdnimg.cn/img_convert/814235dcce5471c2a527e82bafcf21c7.png)

默认上下文加载器就是应用类加载器，这样以上下文加载器为中介，使得启动类加载器中的代码也可以访问应用类加载器中的类。

**第三次破坏双亲委派机制**

双亲委派模型的第三次“被破坏”是由于用户对程序动态性的追求而导致的。如：**代码热替换(Hot Swap)、模块热部署(Hot Deployment)**等

IBM公司主导的JSR-291(即OSGiR4.2)实现模块化热部署的关键是它自定义的类加载器机制的实现，每一个程序模块(osGi中称为Bundle)都有一个自己的类加载器，当需要更换一个Bundle时，就把Bund1e连同类加载器一起换掉以实现代码的热替换。在oSGi环境下，类加载器不再双亲委派模型推荐的树状结构，而是进一步发展为更加复杂的网状结构。

当收到类加载请求时，OSGi将按照下面的顺序进行类搜索：

1）$\color{red}{将以java.*开头的类，委派给父类加载器加载。}$

2）$\color{red}{否则，将委派列表名单内的类，委派给父类加载器加载。}$

3）否则，将Import列表中的类，委派给Export这个类的Bundle的类加载器加载。

4）否则，查找当前Bundle的ClassPath，使用自己的类加载器加载。

5）否则，查找类是否在自己的Fragment Bundle中，如果在，则委派给Fragment Bundle的类加载器加载。

6）否则，查找Dynamic Import列表的Bundle，委派给对应Bund1e的类加载器加载。

7）否则，类查找失败。

说明：只有开头两点仍然符合双亲委派模型的原则，其余的类查找都是在平级的类加载器中进行的

小结：这里，我们使用了“被破坏”这个词来形容上述不符合双亲委派模型原则的行为，但这里“被破坏”并不一定是带有贬义的。只要有明确的目的和充分的理由，突破旧有原则无疑是一种创新。

正如：OSGi中的类加载器的设计不符合传统的双亲委派的类加载器架构，且业界对其为了实现热部署而带来的额外的高复杂度还存在不少争议，但对这方面有了解的技术人员基本还是能达成一个共识，认为**OSGi中对类加载器的运用是值得学习的，完全弄懂了OSGi的实现，就算是掌握了类加载器的精粹。**

## 5.4. 热替换的实现

热替换是指在程序的运行过程中，不停止服务，只通过替换程序文件来修改程序的行为。$\color{red}{热替换的关键需求在于服务不能中断，修改必须立即表现正在运行的系统之中。}$基本上大部分脚本语言都是天生支持热替换的，比如：PHP，只要替换了PHP源文件，这种改动就会立即生效，而无需重启Web服务器。

但对Java来说，热替换并非天生就支持，如果一个类已经加载到系统中，通过修改类文件，并无法让系统再来加载并重定义这个类。因此，在Java中实现这一功能的一个可行的方法就是灵活运用ClassLoader。

注意：由不同ClassLoader加载的同名类属于不同的类型，不能相互转换和兼容。即两个不同的ClassLoader加载同一个类，在虚拟机内部，会认为这2个类是完全不同的。

根据这个特点，可以用来模拟热替换的实现，基本思路如下图所示：

![image-20210501182003439](https://img-blog.csdnimg.cn/img_convert/9ae876265c85af4e431932647993dab7.png)

以下代码在运行时对Demo1进行编译，会立马作用于在运行的程序中！

![QQ_1727421594124](/Users/yannlau/Documents/JavaSet/Java韩顺平/高阶专题/JVM专题/NOTE_JVM_宋红康/JVM中篇：字节码与类的加载篇/04-再谈类的加载器/assets/QQ_1727421594124.png)

<hr/>

# 6. 沙箱安全机制

沙箱安全机制

- 保证程序安全
- 保护Java原生的JDK代码

$\color{red}{Java安全模型的核心就是Java沙箱（sandbox）}$。什么是沙箱？沙箱是一个限制程序运行的环境。

沙箱机制就是将Java代码$\color{red}{限定在虚拟机（JVM）特定的运行范围中，并且严格限制代码对本地系统资源访问}$。通过这样的措施来保证对代码的有限隔离，防止对本地系统造成破坏。

沙箱主要限制系统资源访问，那系统资源包括什么？CPU、内存、文件系统、网络。不同级别的沙箱对这些资源访问的限制也可以不一样。

所有的Java程序运行都可以指定沙箱，可以定制安全策略。

## 6.1. JDK1.0时期

在Java中将执行程序分成本地代码和远程代码两种，本地代码默认视为可信任的，而远程代码则被看作是不受信的。对于授信的本地代码，可以访问一切本地资源。而对于非授信的远程代码在早期的Java实现中，安全依赖于**沙箱（Sandbox）机制**。如下图所示JDK1.0安全模型

![image-20210501182608205](https://img-blog.csdnimg.cn/img_convert/71caf905a0ca13866e24419b7faa14ee.png)

## 6.2. JDK1.1时期

JDK1.0中如此严格的安全机制也给程序的功能扩展带来障碍，比如当用户希望远程代码访问本地系统的文件时候，就无法实现。

因此在后续的Java1.1版本中，针对安全机制做了改进，增加了**安全策略**。允许用户指定代码对本地资源的访问权限。

如下图所示JDK1.1安全模型

![image-20210501182626963](https://img-blog.csdnimg.cn/img_convert/b93f0829e8340131a49738f8843307e4.png)

## 6.3. JDK1.2时期

在Java1.2版本中，再次改进了安全机制，增加了**代码签名**。不论本地代码或是远程代码，都会按照用户的安全策略设定，由类加载器加载到虚拟机中权限不同的运行空间，来实现差异化的代码执行权限控制。如下图所示JDK1.2安全模型：

![image-20210501182652378](https://img-blog.csdnimg.cn/img_convert/81919ed70977f2b57ef5a11481c68abc.png)

## 6.4. JDK1.6时期

当前最新的安全机制实现，则引入了**域（Domain）**的概念。

虚拟机会把所有代码加载到不同的系统域和应用域。$\color{red}{系统域部分专门负责与关键资源进行交互}$，而各个应用域部分则通过系统域的部分代理来对各种需要的资源进行访问。虚拟机中不同的受保护域（Protected Domain），对应不一样的权限（Permission）。存在于不同域中的类文件就具有了当前域的全部权限，如下图所示，最新的安全模型（jdk1.6）

![image-20210501182740197](https://img-blog.csdnimg.cn/img_convert/cf22059daec45f370ae3afb33b870c1f.png)
<hr/>

# 7. 自定义类的加载器

## 7.1. 为什么要自定义类加载器？

- $\color{red}{隔离加载类}$

  在某些框架内进行中间件与应用的模块隔离，把类加载到不同的环境。比如:阿里内某容器框架通过自定义类加载器确保应用中依赖的jar包不会影响到中间件运行时使用的jar包。再比如:Tomcat这类Web应用服务器，内部自定义了好几种类加载器，用于隔离同一个Web应用服务器上的不同应用程序。（类的仲裁-->类冲突）

- $\color{red}{修改类加载的方式}$

  类的加载模型并非强制，除Bootstrap外，其他的加载并非一定要引入，或者根据实际情况在某个时间点进行按需进行动态加载

- $\color{red}{扩展加载源}$

  比如从数据库、网络、甚至是电视机机顶盒进行加载

- $\color{red}{防止源码泄漏}$

  Java代码容易被编译和篡改，可以进行编译加密。那么类加载也需要自定义，还原加密的字节码。

**常见的场景**

- 实现类似进程内隔离，类加载器实际上用作不同的命名空间，以提供类似容器、模块化的效果。例如，两个模块依赖于某个类库的不同版本，如果分别被不同的容器加载，就可以互不干扰。这个方面的集大成者是JavaEE和OSGI、JPMS等框架。
- 应用需要从不同的数据源获取类定义信息，例如网络数据源，而不是本地文件系统。或者是需要自己操纵字节码，动态修改或者生成类型。

**注意**

在一般情况下，使用不同的类加载器去加载不同的功能模块，会提高应用程序的安全性。但是，如果涉及Java类型转换，则加载器反而容易产生不美好的事情。在做Java类型转换时，只有两个类型都是由同一个加载器所加载，才能进行类型转换，否则转换时会发生异常。

## 7.2. 实现方式

Java提供了抽象类java.lang.ClassLoader，所有用户自定义的类加载器都应该继承ClassLoader类。

在自定义ClassLoader的子类时候，我们常见的会有两种做法:

- 方式一:重写loadClass()方法
- 方式二:重写findclass()方法

**对比**

- 这两种方法本质上差不多，毕竟loadClass()也会调用findClass()，但是从逻辑上讲我们最好不要直接修改loadClass()的内部逻辑。建议的做法是只在findClass()里重写自定义类的加载方法，根据参数指定类的名字，返回对应的Class对象的引用。
- loadclass()这个方法是实现双亲委派模型逻辑的地方，擅自修改这个方法会导致模型被破坏，容易造成问题。$\color{red}{因此我们最好是在双亲委派模型框架内进行小范围的改动，不破坏原有的稳定结构}$。同时，也避免了自己重写loadClass()方法的过程中必须写双亲委托的重复代码，从代码的复用性来看，不直接修改这个方法始终是比较好的选择。
- 当编写好自定义类加载器后，便可以在程序中调用loadClass()方法来实现类加载操作。

**说明**

- 其父类加载器默认就是系统类加载器
- JVM中的所有类加载都会使用java.lang.ClassLoader.loadClass(String)接口(自定义类加载器并重写java.lang.ClassLoader.loadClass(String)接口的除外)，连JDK的核心类库也不能例外。

<img src="/Users/yannlau/Documents/JavaSet/Java韩顺平/高阶专题/JVM专题/NOTE_JVM_宋红康/JVM中篇：字节码与类的加载篇/04-再谈类的加载器/assets/QQ_1727426463395.png" alt="QQ_1727426463395" style="zoom:50%;" />

<hr/>

# 8. Java9新特性

JDK9以后

![QQ_1727426804061](/Users/yannlau/Documents/JavaSet/Java韩顺平/高阶专题/JVM专题/NOTE_JVM_宋红康/JVM中篇：字节码与类的加载篇/04-再谈类的加载器/assets/QQ_1727426804061.png)

为了保证兼容性，JDK9没有从根本上改变三层类加载器架构和双亲委派模型，但为了`模块化系统`的顺利运行，仍然发生了一些值得被注意的变动。

1. `扩展机制被移除`，扩展类加载器由于向后兼容性的原因被保留，不过被重命名为平台类加载器(platform class loader)。可以通过classLoader的新方法getPlatformClassLoader()来获取。

   JDK9时基于模块化进行构建(原来的rt.jar和tools.jar被拆分成数十个==JMOD文件==)，其中的Java类库就已天然地满足了可扩展的需求，那自然无须再保`留$JAVA_HOME\lib\ext`目录，此前使用这个目录或者java.ext.dirs系统变量来扩展JDK功能的机制已经没有继续存在的价值了。

2. `平台类加载器和应用程序类加载器都不再继承自java.net.URLClassLoader`。

   现在启动类加载器、平台类加载器、应用程序类加载器全都继承于`jdk.internal.loader.BuiltinClassLoader`。

![img](https://img-blog.csdnimg.cn/img_convert/323cfcda53f98034ed15372c0ea43685.png)

如果有程序直接依赖了这种继承关系，或者依赖了URLClassLoader类的特定方法，那代码很可能会在JDK9及更高版本的JDK中崩溃。

3. 在Java9中，类加载器有了名称。该名称在构造方法中指定，可以通过getName()方法来获取。平台类加载器的名称是platform，应用类加载器的名称是app。类加载器的名称在调试与类加载器相关的问题时会非常有用。
4. `启动类加载器现在是在jvm内部和java类库共同协作实现的类加载器（以前是C++实现），但为了与之前代码兼容，在获取启动类加载器的场景中仍然会返回null，而不会得到BootClassLoader实例。`
5. 类加载的委派关系也发生了变动。当平台及应用程序类加载器收到类加载请求，在委派给父加载器加载前，要先判断该类是否能够归属到某一个系统模块中，如果可以找到这样的归属关系，就要优先委派给负责那个模块的加载器完成加载。

![img](https://img-blog.csdnimg.cn/img_convert/cb23791a5fb1bf1a4c8a28d6a3179e84.png)

![img](https://img-blog.csdnimg.cn/img_convert/ef9b83abcdb9f54d0f0ec7d15f0adc44.png)

![img](https://img-blog.csdnimg.cn/img_convert/192fda50804d35e7d1b44dc61a65ede1.png)

![img](https://img-blog.csdnimg.cn/img_convert/f07a455ec275a6503bfad070ae3d9ffb.png)

**代码：**

```java
public class ClassLoaderTest {
    public static void main(String[] args) {
        System.out.println(ClassLoaderTest.class.getClassLoader());
        System.out.println(ClassLoaderTest.class.getClassLoader().getParent());
        System.out.println(ClassLoaderTest.class.getClassLoader().getParent().getParent());

        //获取系统类加载器
        System.out.println(ClassLoader.getSystemClassLoader());
        //获取平台类加载器
        System.out.println(ClassLoader.getPlatformClassLoader());
        //获取类的加载器的名称
        System.out.println(ClassLoaderTest.class.getClassLoader().getName());
    }
}
```

# 补充

>jdk8之后sun.misc.Launcher没有了，扩展类目录jre/lib/ext 也没有了，到底发生了什么？
>
>###### ChatGPT 说：
>
>在 JDK 8 之后的版本（从 JDK 9 开始），Java 进行了非常大的架构变动，尤其是与模块化系统和类加载机制相关的部分。`sun.misc.Launcher` 及扩展类目录 `jre/lib/ext` 的移除是其中的重要变更之一。以下是具体的解释：
>
>### 1. **模块化系统引入（Jigsaw 项目）**
>
>在 JDK 9 中，Java 引入了模块化系统（**JPMS，Java Platform Module System**），也被称为 **Project Jigsaw**。这一模块化系统重新构建了 JDK，使得 JDK 本身成为模块化的，极大地改变了类加载的结构。
>
>- 在 JDK 8 及之前，Java 的类加载机制中有三种主要的类加载器：
>
>  - **启动类加载器（Bootstrap ClassLoader）**：加载 `rt.jar` 中的核心类库。
>  - **扩展类加载器（Extension ClassLoader）**：加载 `jre/lib/ext` 目录下的类库。
>  - **应用类加载器（Application ClassLoader）**：加载应用程序的类路径下的类。
>
>  从 JDK 9 开始，Jigsaw项目引入了模块化，JDK 本身的类库也被拆分成多个模块（例如，java.base java.xml 等），这些模块不再以 rt.jar 和 jre/lib/ext目录的形式存在，而是通过模块的方式进行组织和加载。
>
>### 2. **`sun.misc.Launcher` 的移除**
>
>在 JDK 8 及之前，`sun.misc.Launcher` 是类加载机制的重要组成部分，它负责启动并设置应用程序的类加载器，包括扩展类加载器和应用类加载器。
>
>- JDK 9 及之后
>
>  ，由于模块系统的引入，类加载器的机制得到了重构：
>
>  - **`sun.misc.Launcher` 不再需要**。它的功能被模块化系统所替代。
>  - 类加载器的设计变得更加模块化和安全，不再依赖于传统的 `sun.misc.Launcher` 进行应用程序的启动。
>
>### 3. **扩展类目录 `jre/lib/ext` 的移除**
>
>- 在 JDK 8 及之前，开发者可以将一些类库放在 `jre/lib/ext` 目录下，这些类库会被扩展类加载器自动加载。在 JDK 9 之后，随着模块化系统的引入，扩展类目录 `jre/lib/ext` 被移除。
>- **模块系统的优点**：
>  - 通过模块系统，Java 提供了更严格的依赖管理和隔离机制，取代了原有的扩展类加载机制。
>  - 开发者不再依赖 `jre/lib/ext`，而是通过模块路径（**module path**）和类路径（**class path**）来指定依赖。
>
>### 4. **JDK 9 之后的类加载器变化**
>
>- **启动类加载器（Bootstrap ClassLoader）**：依然存在，但它现在加载的是模块化系统中的基础模块（如 `java.base`）。
>- **平台类加载器（Platform ClassLoader）**：替代了之前的扩展类加载器，用于加载 JDK 自带的模块，如 `java.xml`、`java.logging` 等。
>- **应用类加载器（Application ClassLoader）**：仍然负责加载应用程序的类路径，但它现在支持加载模块路径中的模块。
>
>### 5. **总结**
>
>- **`sun.misc.Launcher`** 在 JDK 9 及之后被模块化系统取代，不再需要。
>- **`jre/lib/ext` 扩展类目录** 也被模块系统所取代，开发者需要使用模块路径或类路径来管理库和依赖。
>- JDK 9 引入的模块化系统为 Java 提供了更强的封装性、可维护性和安全性，并且重构了类加载的架构。
>
>这些变更使得 Java 在模块化方面变得更加灵活和安全，适应现代化的需求。
>
>##### 您说：
>
>请分别举例说明模块路径、类路径的用法
>
>###### ChatGPT 说：
>
>ChatGPT
>
>在 JDK 9 及之后的 Java 版本中，类路径（**Class Path**）和模块路径（**Module Path**）是用于管理 Java 应用程序依赖的两种不同机制。下面分别举例说明它们的用法。
>
>### 1. **类路径（Class Path）用法**
>
>类路径是 JDK 8 及之前版本中常用的机制，用于指定应用程序所依赖的 `.class` 文件或 JAR 包的路径。它在 JDK 9 及之后仍然可以使用，但模块化系统鼓励使用模块路径。
>
>#### 示例：使用类路径运行 Java 程序
>
>假设有以下文件结构：
>
>```
>/myapp
> ├── src
> │   └── com
> │       └── example
> │           └── Main.java
> └── lib
>     └── external-lib.jar
>```
>
>- `Main.java` 文件内容：
>
>```java
>package com.example;
>
>public class Main {
>    public static void main(String[] args) {
>        System.out.println("Hello, Class Path!");
>    }
>}
>```
>
>**步骤 1**：编译 `Main.java`
>
>```
>javac -d out src/com/example/Main.java
>```
>
>**步骤 2**：使用类路径运行程序
>
>```
>java -cp out:lib/external-lib.jar com.example.Main
>```
>
>这里，`-cp` 选项用于指定类路径，`out` 是编译后的 `.class` 文件目录，`lib/external-lib.jar` 是应用程序所依赖的 JAR 文件。:用于分割路径
>
>#### 解释：
>
>- `-cp` 或 `-classpath` 用于设置 Java 虚拟机搜索类或包的路径。
>- 当使用类路径时，所有类库和依赖需要通过手动配置，Java 没有对类进行模块化管理。
>
>------
>
>### 2. **模块路径（Module Path）用法**
>
>从 JDK 9 开始，Java 引入了模块系统，支持使用模块路径来管理模块化的依赖。模块路径允许你将应用程序和库组织为模块，并通过模块描述符 `module-info.java` 来声明依赖关系。
>
>#### 示例：使用模块路径运行 Java 模块化程序
>
>假设有以下文件结构：
>
>```
>/myapp
> ├── src
> │   ├── com.example
> │   │   ├── module-info.java
> │   │   └── Main.java
> └── mods
>     └── external-module.jar
>```
>
>- `module-info.java` 文件内容：
>
>```
>module com.example {
>    requires external.module;  // 声明对 external.module 模块的依赖
>}
>```
>
>- `Main.java` 文件内容：
>
>```java
>package com.example;
>
>public class Main {
>    public static void main(String[] args) {
>        System.out.println("Hello, Module Path!");
>    }
>}
>```
>
>**步骤 1**：编译模块
>
>```
>javac -d out --module-path mods(mods所在的路径为当前路径下的mods文件夹) src/com/example/module-info.java src/com/example/Main.java
>```
>
>**步骤 2**：使用模块路径运行程序
>
>```
>java --module-path out:mods --module com.example/com.example.Main
>```
>
>#### 解释：
>
>- `--module-path` 用于设置模块路径，模块路径是存放模块（通常为模块化 JAR 文件或模块化的目录）的地方。
>- `module-info.java` 是模块的描述符，定义了模块的名称及其依赖项（如 `requires external.module` 声明了对其他模块的依赖）。
>- `--module` 用于指定要运行的模块和主类。
>
>与类路径不同，模块路径能更好地组织依赖，提供了模块间的强封装和依赖管理。
>
>------
>
>### **类路径和模块路径的对比**
>
>- **类路径**：没有模块化概念，所有类都是在单一的命名空间中，不具备依赖声明的能力。适合非模块化的项目或简单的依赖管理。
>- **模块路径**：支持模块化，允许开发者通过 `module-info.java` 文件显式声明模块的依赖和导出哪些包，提供更强的封装和模块间隔离，适合复杂的、需要依赖管理的大型项目。
>
>使用模块路径的好处是增强了代码组织和运行时安全性，而类路径则适合传统的、无需模块化的项目。
>
>##### 您说：
>
>那么JDK8之后还存在双亲委派机制吗？
>
>###### ChatGPT 说：
>
>ChatGPT
>
>是的，**双亲委派机制**（**Parent Delegation Model**）在 JDK 8 之后仍然存在，即便在引入模块化系统（Jigsaw 项目）之后，JDK 9 及之后的版本依然沿用了这一类加载机制。只是类加载器的结构有所调整，但基本的双亲委派原则仍然有效。
>
>### 1. **什么是双亲委派机制？**
>
>双亲委派机制是一种类加载模型，目的是为了保证 Java 平台的核心类库具有更高的安全性和稳定性，防止用户自定义类加载器加载核心库中的类（如 `java.lang.String`）。具体的加载流程是：
>
>- 当类加载器（无论是系统类加载器还是自定义类加载器）需要加载某个类时，它会首先把请求委托给它的父加载器去尝试加载。
>- 如果父加载器能够找到该类，就直接返回该类。如果父加载器找不到，才会由当前类加载器尝试加载。
>
>在这种机制下，类加载是自上而下进行的，从启动类加载器（`Bootstrap ClassLoader`）到扩展类加载器（`Extension ClassLoader`）再到应用类加载器（`Application ClassLoader`），最后到自定义类加载器。
>
>### 2. **JDK 9 及之后的类加载器变化**
>
>从 JDK 9 开始，引入了模块化系统，JDK 的内部结构发生了很大变化，但双亲委派机制依然保持。以下是 JDK 9 之后的类加载器层次：
>
>- **启动类加载器（Bootstrap ClassLoader）**：依旧存在，用于加载最核心的 Java 类（如 `java.base` 模块中的类）。
>- **平台类加载器（Platform ClassLoader）**：替代了之前的扩展类加载器，负责加载 JDK 自带的模块（如 `java.xml` 模块）。
>- **应用类加载器（Application ClassLoader）**：依然负责加载应用程序的类路径（Class Path）或模块路径（Module Path）中的类。
>
>这些类加载器依然使用双亲委派机制。例如，假设应用程序需要加载 `java.lang.String` 类，应用类加载器不会自己加载，而是委托给父加载器（启动类加载器）来处理。
>
>### 3. **模块化系统与双亲委派的关系**
>
>虽然 JDK 9 引入了模块化系统，但模块化系统并没有改变类加载的基本流程。模块化系统主要改变的是类的可见性和模块之间的依赖管理，而不是类加载器的行为。
>
>具体来说，模块系统引入了**模块路径（Module Path）**，并通过 `module-info.java` 文件来定义模块的依赖关系。然而，模块的类仍然是通过类加载器加载的，只不过类的加载顺序和权限由模块系统进行了进一步的控制和限制。模块化系统通过限制模块之间的可见性来增强类的封装性，但这并不影响类加载器的双亲委派机制。
>
>### 4. **自定义类加载器与双亲委派**
>
>在 JDK 8 及之前，开发者可以通过自定义类加载器改变类的加载方式，JDK 9 及之后依然支持这种做法。双亲委派机制同样适用于自定义类加载器。自定义类加载器在加载类时，也会优先把请求委托给父类加载器，而不是直接加载。
>
>如果你创建了一个自定义类加载器，但希望它不遵循双亲委派（即希望它优先加载某些类），你可以在自定义的 `ClassLoader` 中覆盖 `findClass` 或 `loadClass` 方法。不过，破坏双亲委派机制会带来潜在的风险，尤其可能导致与核心类的冲突。
>
>### 5. **总结**
>
>- **双亲委派机制在 JDK 8 之后依然存在**，即便在模块化系统引入后，它仍然是类加载过程的基础。
>- **JDK 9 模块化系统** 的引入更多的是改变了类的可见性和模块间的依赖管理，而不是替换或改变双亲委派机制。
>- **类加载器结构**在 JDK 9 之后有所调整，扩展类加载器被平台类加载器取代，但双亲委派模型依然保持。
>
>
>
>##### 您说：
>
>我使用自定义类加载器加载了一个指定文件夹内的User.class文件，其中的User类也是public访问权限的，但是实际上我并不能在加载之后直接在接下来的程序中写User user = new User();。为什么只有import之后才可以？
>
>
>
>###### ChatGPT 说：
>
>ChatGPT
>
>你遇到的问题与**类加载器的命名空间和类的可见性**有关。以下是解释为什么你需要 `import` 才能使用自定义类加载器加载的类。
>
>### 1. **类加载器的命名空间**
>
>在 Java 中，不同的类加载器有各自独立的**命名空间**。即使两个类具有相同的全限定类名（例如 `com.example.User`），如果它们是由不同的类加载器加载的，JVM 会将它们视为**不同的类**。
>
>你提到的情况是，自定义类加载器加载了 `User.class`，但你不能直接使用它，因为：
>
>- 在你的代码中，默认的类加载器（如系统类加载器）没有加载这个 `User` 类。
>- 虽然你定义了一个 `User` 类，并且这个类是 `public`，但是它是由你自定义的类加载器加载的，而不是由系统类加载器或应用类加载器加载的。
>
>Java 中，类加载器之间不会共享彼此加载的类，除非你显式地处理它们的关系。因此，如果你不通过 `import` 或指定正确的类加载器，默认情况下，代码并不知道如何找到由自定义类加载器加载的 `User` 类。
>
>### 2. **为什么需要 `import`？**
>
>`import` 是 Java 中用于声明类或包的机制，它告诉编译器在哪个包或类路径下找到指定的类。在你使用自定义类加载器加载 `User` 类时，编译器需要知道该类的存在。
>
>当你加载一个类时，即使它是 `public`，编译器需要解析类名。由于类是由自定义类加载器加载的，编译器并不会自动将该类与当前的命名空间中的 `User` 进行关联。因此，只有通过 `import` 明确地将这个类引入，才能让编译器知道如何找到它。
>
>### 3. **自定义类加载器的工作机制**
>
>当你用自定义类加载器加载一个类时，实际上是在运行时动态加载它。即使这个类在运行时已经存在，它并没有自动被当前的类路径或系统类加载器所感知。你需要确保以下几点：
>
>- **编译时问题**：如果你在编译期没有导入这个类，编译器会不知道这个类的存在，因此会报错。`import` 是为了告诉编译器需要使用哪个包中的类。
>- **类加载问题**：你的自定义类加载器在加载类时，它的加载行为与系统类加载器是分开的。因此，JVM 认为这个 `User` 类和通过系统类加载器加载的其他类是**不同的类**，即使它们的类名完全相同。
>
>### 4. **类的冲突与 ClassCastException**
>
>如果你试图通过不同的类加载器加载相同类（同一个 `User.class`），可能会引发 `ClassCastException` 错误。Java 会认为通过自定义类加载器加载的 `User` 类和通过系统类加载器加载的 `User` 类是**不同的类**，因为它们来自不同的类加载器，即使它们的名字、包名和内容完全一样。
>
>```java
>User user = new User();  // User might be loaded by the system class loader
>
>User customUser = (User) customClassLoader.loadClass("User").newInstance();  // Loaded by a custom class loader
>
>// ClassCastException: cannot cast User to User, because they are from different class loaders
>```
>
>在这种情况下，即使类的定义完全相同，Java 也会认为它们是**不同的类型**，因为它们来自不同的类加载器。
>
>### 5. **解决方案**
>
>你可以使用以下方法来避免此类问题：
>
>1. **确保类加载器的一致性**：尽量让自定义类加载器委托给系统类加载器，以避免加载两个不同的 `User` 类。
>   - 通过双亲委派机制，尽可能使用系统类加载器去加载公共类，减少类加载器之间的冲突。
>2. **正确使用 `import`**：当你知道某个类是通过自定义类加载器加载时，确保在编译期导入它。编译器需要明确知道这个类的存在，才能在后续的代码中使用它。
>3. **避免同名类的冲突**：确保你在自定义类加载器中加载的类不会与系统类加载器加载的类发生冲突。
>
>### 6. **总结**
>
>- `User user = new User();` 之所以不能直接使用，是因为编译器在编译时并不知道你自定义类加载器加载的 `User` 类。如果你没有通过 `import` 明确引入该类，编译器无法找到它。
>- 自定义类加载器的命名空间和系统类加载器是分开的，导致即使两个类名字相同，它们在 JVM 中也是不同的类。
>- 通过 `import` 或确保类加载器的一致性，能够解决这个问题。

> 那么如果我用系统类加载器加载User,可不可以在接下来直接声明，而不import导入，也就是会所不在classpath中指定类路径
>
> ###### ChatGPT 说：
>
> 如果你使用**系统类加载器**（即 `Application ClassLoader` 或默认的类加载器）加载 `User` 类，而 `User` 类**没有在类路径（classpath）中指定**，那么：
>
> 1. **在编译时**，你将无法直接声明 `User user = new User();`，因为编译器找不到该类。这是因为在 Java 中，编译器需要通过类路径或导入的包来定位和解析所有类。如果类不在类路径中，编译器无法访问它，因此即使你在运行时通过类加载器加载了 `User.class`，也无法在编译时使用这个类。
> 2. **在运行时**，即便你动态加载了 `User.class`（通过反射或自定义类加载器），编译器仍然需要在编译时知道类的定义。Java 是静态类型语言，在编译时必须解析所有的类引用。因此，**你仍然需要导入类或确保它在类路径中**，以便编译器能够识别和处理该类。
>
> ### 总结：
>
> - **编译时**：如果 `User.class` 不在类路径中，且没有通过 `import` 导入，编译器将无法解析 `User` 类，导致编译错误。
> - **运行时**：即使你在运行时通过系统类加载器成功加载了 `User.class`，也不能在源代码中直接声明 `User user = new User();`，因为编译器在编译时需要知道该类的定义。
>
> ### 解决办法：
>
> 1. **确保 `User.class` 在类路径中**： 将 `User.class` 所在的目录或 JAR 文件添加到类路径中。例如：
>
>    ```bash
>    javac -cp /path/to/user/classes:. Main.java
>    java -cp /path/to/user/classes:. Main
>    ```
>
> 2. **动态加载类并通过反射使用**： 如果你确实想在运行时加载类，而不在编译时导入类，可以使用反射来创建类的实例，并调用它的方法。示例如下：
>
>    ```java
>    // 动态加载 User 类
>    Class<?> userClass = Class.forName("com.example.User");
>    Object userInstance = userClass.getDeclaredConstructor().newInstance();
>    
>    // 使用反射调用 User 类的方法
>    Method method = userClass.getMethod("someMethod");
>    method.invoke(userInstance);
>    ```
>
> 通过这种方式，你可以绕过编译时的类解析问题，但你需要通过反射来动态操作对象，而不能直接使用 `User user = new User();` 这样的静态声明。
>
> 
>
> ##### 您说：
>
> 也就说想直接使用必须要import导入？
>
> ###### ChatGPT 说：
>
> 是的，**要直接使用一个类**（例如 `User user = new User();`）必须确保该类在编译时对编译器可见，而这通常需要通过以下两种方式之一：
>
> 1. **通过类路径（classpath）**：`User.class` 文件必须位于类路径中，这样编译器在编译时能够找到这个类。
> 2. **通过 `import` 导入**：如果类在类路径中，但位于不同的包内，你还需要在源代码中使用 `import` 语句导入该类，确保编译器知道你要使用哪个具体的类。
>
> ### 为什么必须 `import`？
>
> 1. **编译期要求**：Java 是静态类型语言，编译器在编译时需要知道所有使用的类的定义。`import` 告诉编译器从哪里找到指定的类，或者在当前包内自动识别该类。
> 2. **命名空间管理**：`import` 用来处理包的命名空间问题。即使类在类路径中可见，如果它位于不同的包中，你仍然需要使用 `import` 语句来告知编译器使用哪个包下的类，否则只能通过全限定类名来使用（例如 `com.example.User user = new com.example.User();`）。
>
> ### 特殊情况：
>
> - **同一个包内的类**：如果 `User` 类和你的当前类位于同一个包内，那么不需要使用 `import`，可以直接使用 `User user = new User();`。这是因为编译器会自动在当前包下查找类。
> - **反射机制**：如果你不想在编译时明确导入类，也可以通过**反射**机制在运行时动态加载类，如前面提到的方式。但即便如此，你仍然不能直接用常规语法（如 `User user = new User();`），必须通过反射来操作。
>
> ### 示例：
>
> ```java
> // 假设 User 在 com.example 包中
> import com.example.User;
> 
> public class Main {
>     public static void main(String[] args) {
>         User user = new User();  // 直接使用
>     }
> }
> ```
>
> 如果没有 `import com.example.User;`，则编译器不知道哪个包中的 `User` 类，编译会失败。
>
> ### 结论：
>
> 是的，**要直接使用类，必须确保类路径中可见，并且如果它在不同的包中，则需要使用 `import` 导入**。
>
> ##### 您说：
>
> 我个人认为原因应该是：代码中使用类加载器加载的类而不用import导入，因为在代码的编译阶段仅仅只是编译，而不是执行，所以代码中累加器加载类的逻辑是无法被编译器知道的，因此也就无法使用，对吗？
>
> ###### ChatGPT 说：
>
> ChatGPT
>
> 是的，你的理解是正确的。编译器在**编译阶段**只检查代码中的类引用是否可见，而不执行代码，因此它无法知道**运行时**通过类加载器动态加载的类。这是因为编译器的工作仅限于静态代码分析，运行时的行为（例如类加载器加载类的逻辑）只有在代码执行时才会发生。因此，编译器无法在编译阶段识别和使用这些类。
>
> ### 更具体的原因：
>
> 1. **编译阶段 vs 运行阶段**：
>    - **编译阶段**：编译器的任务是将源代码转换为字节码，它会检查所有的类引用、变量、方法等是否定义正确。此时，编译器依赖于类路径（classpath）和 `import` 语句来找到并解析类的定义。如果类在编译时不可见（即不在类路径中或没有导入），编译器会报错。
>    - **运行阶段**：在程序运行时，JVM 才会通过类加载器（包括你可能自定义的类加载器）来加载需要的类。这些类在编译时并没有被实际加载，只是在运行时通过类加载器动态引入。
> 2. **类加载器的作用是在运行时**：
>    - 类加载器是 JVM 在运行时用于加载类的机制。即使你通过类加载器加载了 `User` 类（例如通过 `Class.forName()` 或自定义类加载器），这也是在程序运行时发生的，而不是在编译时。
>    - 编译器无法提前知道运行时会加载哪些类，因此即使你在代码中写了类加载器的逻辑，编译器仍然不能识别这些类，除非它们在编译时通过 `import` 或类路径可见。
>
> ### 举例说明：
>
> 假设你有以下代码：
>
> ```java
> public class Main {
>     public static void main(String[] args) throws Exception {
>         ClassLoader classLoader = Main.class.getClassLoader();
>         Class<?> userClass = classLoader.loadClass("com.example.User");
>         
>         // Create an instance of the dynamically loaded class
>         Object userInstance = userClass.getDeclaredConstructor().newInstance();
>     }
> }
> ```
>
> 这里，`User.class` 是通过类加载器在**运行时**加载的。但是：
>
> - **编译时**：编译器不会执行 `classLoader.loadClass("com.example.User")` 这段代码，因此编译器根本无法知道 `User` 类的存在。
> - **运行时**：当程序实际执行到这段代码时，`User.class` 才会通过类加载器加载到 JVM 中，此时才可以使用反射等方式创建 `User` 类的实例。
>
> 因此，在编译阶段，`User user = new User();` 这样的静态声明是不可行的，编译器会报错，因为它无法找到 `User` 类，除非你在类路径中包含了 `User.class` 或通过 `import` 导入。
>
> ### 总结：
>
> - 你不能在编译阶段直接使用类加载器动态加载的类，因为编译器无法知道这些类的存在。
> - 类加载器的加载过程发生在运行时，而编译器只在编译阶段处理静态信息（如类路径和 `import`），因此需要使用 `import` 或类路径来确保类在编译时是可见的。
