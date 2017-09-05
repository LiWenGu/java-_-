本章主要内容：
1. String的例子，见证下我们的功底
2. 一些简单算法你会如何理解
3. 是简单数字游戏玩一玩
4. 功底概述
5. 原生态类型其他例子
6. 集合类的那些破事儿
7. 常见的工具包
8. 面对技术，我们纠结的那些事儿
9. 老A师在逆境中迎难而上者  

---

# 1. String的例子，见证下我们的功底
代码如下：
```java
private static void testStr() {
    String a = "a" + "b" + 1;
    String b = "ab1";
    System.out.println(a == b);
}
```
结果为true，为什么呢？要理解这个问题，你需要了解：
1. 关于“==”是做什么的？
2. equals呢？
3. a和b在内存中是什么样的？
4. 编译时优化方案。

## 1.1 关于“==”
在Java语言中，当“==”匹配的时候，对比的是两个内存单元的内容是否一样。

如果是原始类型byte、boolean、short、char、int、long、float、double，则直接比较值。

如果是引用（Reference），比较的就是引用的值，“引用的值”可以被认为是对象的逻辑地址。即比较两个对象的地址值是否一样。换一句话说，如果两个引用所保存的对象是同一个对象，则返回true。（如果引用指向的是null，即也为true）。

## 1.2 关于“equals()”
在Object类中的源码如下：

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```
因此，如果没有重写过equals()方法，那么默认的equals()操作就是对比对象的地址。
String就自己实现了equals()方法。主要用于两个对象根据业务的关键属性值来对比，确定它们是否是“一致性的或相似的”，返回true|false即可。

那么equals\(\)重写后，一般会重写hashCode\(\)方法吗？

Java中的hashCode是什么————hashCode()方法提供了对象的hashCode值，它与equals()一样在Object类中提供，不过它是一个native方法，它的返回值默认与`System.identityHashCode(object)`一致。在通常情况下，这个值是对象头部的一部分二进制位组成的数字，这个数字具有一定的标识对象的意义，但绝不等价于地址。

hashCode的作用————它为了产生一个可以标识对象的数字，不论如何复杂的一个对象都可以用一个数字来标识。为什么需要用一个数字来标识对象呢？因为想将对象用在算法中，如果不这样，许多算法还得自己去组装数字，因为算法的基础是建立在数字基础之上。那么对象如何用在算法中呢？

例如，在`HashMap`、`HashSet`等类似的集合类中，如果用某个对象本身作为Key，也就是要基于

