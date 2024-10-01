## 1-解析字节码的作用

通过反编译生成的字节码文件，我们可以深入的了解java代码的工作机制。但是，自己分析类文件结构太麻烦了！除了使用第三方的jclasslib工具之外，oracle官方也提供了工具：javap。

javap是jdk自带的反解析工具。它的作用就是根据class字节码文件，反解析出当前类对应的code区（字节码指令）、局部变量表、异常表和代码行偏移量映射表、常量池等信息。

通过局部变量表，我们可以查看局部变量的作用域范围、所在槽位等信息，甚至可以看到槽位复用等信息。|

## 2-javac-g操作
解析字节码文件得到的信息中，有些信息（如局部变量表、指令和代码行偏移量映射表、常量池中方法的参数名称等等）需要在使用javac编译成class文件时，指定参数才能输出。

比如，你`直接javac xx.java，就不会在生成对应的局部变量表等信息`，如果你使用`javac -g xx.java`就可以生成所有相关信息了。如果你使用的eclipse或IDEA，则`默认情况下，eclipse、IDEA在编译时会帮你生成局部变量表、指令和代码行偏移量映射表等信息的。`

## 3-javap的用法
javap的用法格式：

`javap ‹options› ‹classes›`

其中，classes 就是你要反编译的class文件。

在命令行中直接输入javap 或 javap -help 可以看到javap的options有如下选项： 

```java
用法: javap <options> <classes>
其中, 可能的选项包括:
  --help -help -h -?               输出此帮助消息
  -version                         版本信息
  -v  -verbose                     输出附加信息
  -l                               输出行号和本地变量表
  -public                          仅显示公共类和成员
  -protected                       显示受保护的/公共类和成员
  -package                         显示程序包/受保护的/公共类
                                   和成员 (默认)
  -p  -private                     显示所有类和成员
  -c                               对代码进行反汇编
  -s                               输出内部类型签名
  -sysinfo                         显示正在处理的类的
                                   系统信息（路径、大小、日期、SHA-256 散列）
  -constants                       显示最终常量
  --module <模块>, -m <模块>       指定包含要反汇编的类的模块
  -J<vm-option>                    指定 VM 选项
  --module-path <路径>             指定查找应用程序模块的位置
  --system <jdk>                   指定查找系统模块的位置
  --class-path <路径>              指定查找用户类文件的位置
  -classpath <路径>                指定查找用户类文件的位置
  -cp <路径>                       指定查找用户类文件的位置
  -bootclasspath <路径>            覆盖引导类文件的位置
  --multi-release <version>        指定要在多发行版 JAR 文件中使用的版本

GNU 样式的选项可使用 = (而非空白) 来分隔选项名称
及其值。

每个类可由其文件名, URL 或其
全限定类名指定。示例:
   path/to/MyClass.class
   jar:file:///path/to/MyJar.jar!/mypkg/MyClass.class
   java.lang.Object
```

`javap -v -p xxx.class` 输出信息最全

一般常用的是 -v -l -c 三个选项。

javap -l 会输出行号和本地变量表信息。

javap -c 会对当前class字节码进行反编译生成汇编代码（方法操作的字节码）。

javap -v classxx 除了包含-c内容外，还会输出行号、局部变量表信息、常量池等信息。

```java
Classfile /Users/yannlau/Desktop/TestForJNI/NativeExample.class //字节码文件所属的绝对路径
  Last modified 2024年9月25日; size 877 bytes  //最后修改时间，字节码文件的大小
  SHA-256 checksum 79dd8cbd80b79dac1a98262f9f4096dead0a9a4277cdc8fd232726b7bd39adf0  //SHA-256散列值
  Compiled from "NativeExample.java"  //源文件的名称
public class NativeExample
  minor version: 0                    //副版本
  major version: 61										//主版本
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER 			//访问标识
  this_class: #7                          // NativeExample
  super_class: #2                         // java/lang/Object
  interfaces: 0, fields: 4, methods: 4, attributes: 1
Constant pool:
   #1 = Methodref          #2.#3          // java/lang/Object."<init>":()V
   #2 = Class              #4             // java/lang/Object
   #3 = NameAndType        #5:#6          // "<init>":()V
   #4 = Utf8               java/lang/Object
   #5 = Utf8               <init>
   #6 = Utf8               ()V
   #7 = String             #8             // 我是来自Java的私有String字段
   #8 = Utf8               我是来自Java的私有String字段
   #9 = Fieldref           #10.#11        // NativeExample.message:Ljava/lang/String;
  #10 = Class              #12            // NativeExample
  #11 = NameAndType        #13:#14        // message:Ljava/lang/String;
  #12 = Utf8               NativeExample
  #13 = Utf8               message
  #14 = Utf8               Ljava/lang/String;
  #15 = Double             12.34d
  #17 = Fieldref           #10.#18        // NativeExample.d:D
  #18 = NameAndType        #19:#20        // d:D
  #19 = Utf8               d
  #20 = Utf8               D
  #21 = Methodref          #22.#23        // java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
  #22 = Class              #24            // java/lang/Integer
  #23 = NameAndType        #25:#26        // valueOf:(I)Ljava/lang/Integer;
  #24 = Utf8               java/lang/Integer
  #25 = Utf8               valueOf
  #26 = Utf8               (I)Ljava/lang/Integer;
  #27 = Fieldref           #10.#28        // NativeExample.i:Ljava/lang/Integer;
  #28 = NameAndType        #29:#30        // i:Ljava/lang/Integer;
  #29 = Utf8               i
  #30 = Utf8               Ljava/lang/Integer;
  #31 = Fieldref           #10.#32        // NativeExample.s:S
  #32 = NameAndType        #33:#34        // s:S
  #33 = Utf8               s
  #34 = Utf8               S
  #35 = Methodref          #10.#3         // NativeExample."<init>":()V
  #36 = Methodref          #10.#37        // NativeExample.printMessage:()V
  #37 = NameAndType        #38:#6         // printMessage:()V
  #38 = Utf8               printMessage
  #39 = String             #12            // NativeExample
  #40 = Methodref          #41.#42        // java/lang/System.loadLibrary:(Ljava/lang/String;)V
  #41 = Class              #43            // java/lang/System
  #42 = NameAndType        #44:#45        // loadLibrary:(Ljava/lang/String;)V
  #43 = Utf8               java/lang/System
  #44 = Utf8               loadLibrary
  #45 = Utf8               (Ljava/lang/String;)V
  #46 = Utf8               Code
  #47 = Utf8               LineNumberTable
  #48 = Utf8               LocalVariableTable
  #49 = Utf8               this
  #50 = Utf8               LNativeExample;
  #51 = Utf8               main
  #52 = Utf8               ([Ljava/lang/String;)V
  #53 = Utf8               args
  #54 = Utf8               [Ljava/lang/String;
  #55 = Utf8               <clinit>
  #56 = Utf8               SourceFile
  #57 = Utf8               NativeExample.java
{																									
  =====================================字段表集合的信息===============================================
  private java.lang.String message;								//字段名
    descriptor: Ljava/lang/String;								//字段描述符：字段的类型
    flags: (0x0002) ACC_PRIVATE										//字段访问标识

  double d;
    descriptor: D
    flags: (0x0000)

  protected java.lang.Integer i;
    descriptor: Ljava/lang/Integer;
    flags: (0x0004) ACC_PROTECTED

  public short s;
    descriptor: S
    flags: (0x0001) ACC_PUBLIC

      ====================================方法表信息==================================================
  public NativeExample();   						//构造器方法
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=3, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #7                  // String 我是来自Java的私有String字段
         7: putfield      #9                  // Field message:Ljava/lang/String;
        10: aload_0
        11: ldc2_w        #15                 // double 12.34d
        14: putfield      #17                 // Field d:D
        17: aload_0
        18: sipush        1234
        21: invokestatic  #21                 // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        24: putfield      #27                 // Field i:Ljava/lang/Integer;
        27: aload_0
        28: sipush        12345
        31: putfield      #31                 // Field s:S
        34: return
      LineNumberTable:
        line 1: 0
        line 3: 4
        line 4: 10
        line 5: 17
        line 6: 27
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      35     0  this   LNativeExample;

  public native void printMessage();              //Native方法
    descriptor: ()V
    flags: (0x0101) ACC_PUBLIC, ACC_NATIVE

  public static void main(java.lang.String[]);     
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1					//stack：操作数栈的最大深度 locals：局部变量表的长度 arg_size方法参数（非静态方法至少有一个参数this）
      //偏移量 操作码    操作数
         0: new           #10                 // class NativeExample
         3: dup
         4: invokespecial #35                 // Method "<init>":()V
         7: invokevirtual #36                 // Method printMessage:()V
        10: return
      //行号表  指明 字节码指令的偏移量 和 源程序代码的行号 的对应关系
      LineNumberTable:
        line 17: 0
        line 18: 10
     //局部变量表
      LocalVariableTable:
        //Start Length 指明内部局部变量的作用范围
        Start  Length  Slot  Name   Signature
            0      11     0  args   [Ljava/lang/String;

  //clinit
  static {};
    descriptor: ()V
    flags: (0x0008) ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: ldc           #39                 // String NativeExample
         2: invokestatic  #40                 // Method java/lang/System.loadLibrary:(Ljava/lang/String;)V
         5: return
      LineNumberTable:
        line 13: 0
        line 14: 5
}
   //附加属性
SourceFile: "NativeExample.java"  
```

## 5-总结
1、通过javap命令可以查看一个java类反汇编得到的Class文件版本号、常量池、访问标识、变量表、指令代码行号表等等信息。不显示类索引、父类索引、接口索引集合、<clinit>()、<init>()等结构

2、通过对前面例子代码反汇编文件的简单分析，可以发现，一个方法的执行通常会涉及下面几块内存的操作：

（1） java栈中：局部变量表、操作数栈。
（2） java堆。通过对象的地址引用去操作。
（3） 常量池。
（4） 其他如帧数据区、方法区的剩余部分等情况，测试中没有显示出来，这里说明一下。

3、平常，我们比较关注的是java类中每个方法的反汇编中的指令操作过程，这些指令都是顺序执行的，可以参考官方文档查看每个指令的含义，很简单：

https://docs.oracle.com/iavase/specs/ivms/se7/htm1/ivms-6.html