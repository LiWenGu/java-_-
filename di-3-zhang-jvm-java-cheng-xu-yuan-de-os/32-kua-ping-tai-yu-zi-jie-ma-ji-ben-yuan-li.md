注：由于本章讲解性文字较多，因此只摘要了笔者觉得重要的知识，像 `ClassLoader` 加载机制（[点击我详看](http://www.liwenguang.cn/2017/08/10/deepknowjavaweb/6_%E6%B7%B1%E5%85%A5%E5%88%86%E6%9E%90ClassLoader%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6.html/)），还有字节码文件等没有具体的讲解。

## 2.1 `javap` 命令工具

代码如下：

```java
private static void test1() {
    String a = "a" + "b" + 1;
    String b = "ab1";
    System.out.println(a == b);
}
```

使用命令对其生成的字节码文件进行论证： `javap -verbose StringTest.class`

```class
PS C:\Users\liwenguang\Desktop> javap -verbose .\StringTest.class
Classfile /C:/Users/liwenguang/Desktop/StringTest.class
  Last modified 2017-9-18; size 535 bytes
  MD5 checksum 7d584c2a6eeb18b0a7483a45fbb5c1b4
  Compiled from "StringTest.java"
public class StringTest
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#19         // java/lang/Object."<init>":()V
   #2 = String             #20            // ab1
   #3 = Fieldref           #21.#22        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Methodref          #23.#24        // java/io/PrintStream.println:(Z)V
   #5 = Class              #25            // StringTest
   #6 = Class              #26            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               main
  #12 = Utf8               ([Ljava/lang/String;)V
  #13 = Utf8               StackMapTable
  #14 = Class              #27            // "[Ljava/lang/String;"
  #15 = Class              #28            // java/lang/String
  #16 = Class              #29            // java/io/PrintStream
  #17 = Utf8               SourceFile
  #18 = Utf8               StringTest.java
  #19 = NameAndType        #7:#8          // "<init>":()V
  #20 = Utf8               ab1
  #21 = Class              #30            // java/lang/System
  #22 = NameAndType        #31:#32        // out:Ljava/io/PrintStream;
  #23 = Class              #29            // java/io/PrintStream
  #24 = NameAndType        #33:#34        // println:(Z)V
  #25 = Utf8               StringTest
  #26 = Utf8               java/lang/Object
  #27 = Utf8               [Ljava/lang/String;
  #28 = Utf8               java/lang/String
  #29 = Utf8               java/io/PrintStream
  #30 = Utf8               java/lang/System
  #31 = Utf8               out
  #32 = Utf8               Ljava/io/PrintStream;
  #33 = Utf8               println
  #34 = Utf8               (Z)V
  ......
```

### 2.1.1 讲解 `#1` `#6` `#19` `#26` `#7` `#8`

入口位置 \#1 ，代表一个方法入口，由入口 \#6 和 入口 \#19 组成。而 \#6 为一个 `class` ， `class` 是一种引用，它引用了入口 \#26 的常量池。入口 \#19 代表一个表示名称和类型，其中 \#7 代表构造函数，\#8 代表没有入口参数，返回值为 void 。  
即， `java.lang.Object` 类的构造函数方法，入口参数个数为 0 ，返回值为 `void` ，其实这在 `#1` 后面的备注已经标识出来了（这是 `javap` 工具的帮助）。

### 2.1.2 讲解 `#2`

`#2` 代表引用了入口 `#22` 的内容，即常量池存放内容 ab1 。即，一个 `String` 对象的常量，存放到值是 ab1 。

### 2.1.3 讲解 `#3`

入口 `#3` 代表一个属性，这个属性引用了入口 `#21` 的类、入口 \#22 的具体属性。而 `#34` 代表入口参数为 `boolean` 类型。

### 2.1.4 简单数字叠加

```java
public static void test() {
  int a = 1, b = 1, c = 1, d = 1;
  a++;
  ++b;
  c = c++;
  d = ++d;
  System.out.println(a + ", " + b + ", " + c + ", " + d);
}
```

另外，其中也有个问题可以让读者探讨下（值得探讨），[点我进入](https://segmentfault.com/q/1010000011230388)。

## 2.2 `Java` 字节码结构

上一节，我们看到了虚指令，其实是 `javap` 命令帮我们翻译过来的，在 `Java` 程序编译为 `Class` 文件后，它内部的结构是什么样子的呢？

`javac` 命令本身是一个“引导器”，它引导编译器程序的运行。编译器本身也是一个 `Java` 程序当运行 `javac` 命令时所引导的 `Java` 类是 `com.sun.tools.javac.main.JavaCompile` ，这个类会完成 `Java` 源文件的解析（ `Parser` ）、注解处理（ `Annotation process` ）、属性标注、检查、泛型处理、一些语法糖转换，最终生成 `Class` 文件。

首先认识 `Java` 字节码文件的主体结构：  
1. `Class` 文件头部。  
2. 常量池区域。  
3. 属性列表。  
4. 方法列表。  
5. attrbute列表。

`Java` 虚拟机也会为每个 `OS` 平台编写对应的 `JRE` 运行时环境，与 `OS` 动态链接，将这些虚指令编码翻译为对应操作系统的汇编指令信息，即可在对应的 `OS` 上调用执行了。  
1. 为何要有常量池？将内容直接放在字节码中不就行了吗？  
既然叫常量池，那么就是一个信息共享池，其实在真正的程序中，会反反复复出现很多重复的内容拷贝，将它们用常量池保存起来，在描述程序的过程中用很短的一个入口地址就可以表示了，而不需要使用很长的字符串来表示。既可以节约空间，又可以让字节码结构变得更加规整。  
2. 字节码结构，除了跨平台，还有什么用途？   
从两个角度来看，其一，让我们了解到什么是真正的字节码， `Java` 程序编译为 `Class` 字节码文件后到底是什么样子的。了解字节码的结构，这对于我们去了解 `Class` 的内存结构、字节码增强技术、反编译技术都是一个很大的帮助；其二，如果想要设计一种语言或一些类似于解析类的程序，这种字节码结构就是一种可以借鉴的方法。

## 2.3 `Class` 字节码的加载

用到类的时候再加载，是不是会很慢？  
如果是在容器下写代码，大部分时候我们不用关心，大部分类都会被直接或间接加载。另外，在同一个 `ClassLoader` 中的一个类原则上只会被加载一次（热部署，就会被不同的 `ClassLoader` 加载多次）。

1. 同一个“进程”内任何一个 `ClassLoader` 加载的 `Class` 都可以由任意一段程序直接或间接地访问到。在常见的 `Web` 容器（如 `Tomcat` ）中加载不同的 `deploy` 会进行相互隔离，它使用了不同的 `ClassLoader` 来加载不同的 `deploy` ，也就是并非真正意义上的隔离。要想跨 `ClassLoader` 访问一些信息时比较麻烦的，不过也并非不可能，例如使用 `JMX` 就可以帮我们做到。  
2. 在 `Class` 加载中，对于同名的类（也就是 `package` 路劲以及类名都是一样的），在同一个 `ClassLoader` 中只会被加载一次（除非被卸载或重定义）。  
3. `JVM` 做 `FULL GC` 的回收，允许对 `Class` 执行 `unload` 操作，但是苛刻的条件是相应的 `ClassLoader` 下所有的 `Class` 都没有实例的引用，否则这个 `ClassLoader` 及它下面所有的 `Class` 都释放不掉。  
4. 如果希望任意一段代码中所创建的对象是通过自定义的 `ClassLoader` 创建的，那么需要这段代码的调用者所在类是通过相应的 `ClassLoader` 加载的，因此调用者通常需要指定 `ClassLoader` 去实例化才能达到这样的效果。  
5. `ClassLoader` 本身也是一个 `Java` 类，因此它本身也需要加载，是由 `JVM` 内在级别的 `ClassLoader` 开始的。  
6. 使用 `getClass().getResource("").getPath()` 获得这个 `Class` 的来源路径是哪个 `jar` 包。

## 2.4 字节码增强
`AOP` 技术其实是字节码增强技术， `JVM` 提供的动态代理底层也是字节码增强技术。笔者个人觉得书中讲解性比较多，初学者可以见原书此节，这里就不再讲述了，这里推荐一个开源项目：`tiny-spring`
>tiny-spring是为了学习Spring的而开发的，可以认为是一个Spring的精简版。Spring的代码很多，层次复杂，阅读起来费劲。我尝试从使用功能的角度出发，参考Spring的实现，一步一步构建，最终完成一个精简版的Spring。有人把程序员与画家做比较，画家有门基本功叫临摹，tiny-spring可以算是一个程序的临摹版本-从自己的需求出发，进行程序设计，同时对著名项目进行参考。项目地址：https://github.com/code4craft/tiny-spring


