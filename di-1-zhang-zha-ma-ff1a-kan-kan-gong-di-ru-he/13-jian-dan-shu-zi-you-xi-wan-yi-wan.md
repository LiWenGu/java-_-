# 讲一些运算符，然后再讲讲“大数字”是如何处理的。

---

## 3.1 变量 A 、 B 交换有几种方式

1. 使用中间变量
2. 数据叠加后再减回来：
```java
A = A + B;
B = A - B;
A = A - B;
```
3. 异或运算：
```java
A = A ^ B;
B = A ^ B;
A = A ^ B;
```

## 3.2 将无序数据 Hash 到指定的板块
假如有一个长度为5000的数组，每个数组像 Hash 桶那样存放一个链表，此时有一个数据 A 需要写入，随机吗？肯定不行，因为我们还要查询数据。那么将它放在 Hash 桶里面可以吗？似乎可以，下面一起来探讨一下。  
  
我们希望写入的数据能按照某种规则读取出来，那么数据应当与数组额下标产生某种关系，这样写入和读取只要按照相同的规则就可以达到目的。此时假设一个“非负数”要写入，或许可以这样做，将这个数据 % 5000 就可以得到下标了，这样在数字不断变化时，得到的下标也会不断变化，数据自然会相对均匀分布到 5000 个数组中。如果是负数，则需要采用取绝对值 `Math.abs(A)` % 5000 来得到下标。  
  
或者使用高效率的与运算，使用 A & 4999。只是会有个问题：使用数字 11 来说明更清晰，长度为 11 的数组，使用 10 做 & 操作， 10 的二进制位是“1010”，那么意味着 0001 、 0100 、 0101 这些数字将永远是下标为 0 的桶，即， 1 、 8 、 9 这几个下标永远没有数据，因为数据都去下标 0 的桶了。

## 3.3 大量判定“是|否”操作
在“ `java.lang.reflect.Modifier` ”中，它对类型的修饰符有各种各样的判定（例如，字节码中存储 `public final static` 这样的一长串内容只用一个数字来表达），源码如下：
```java
public static final int PUBLIC           = 0x00000001;

public static final int PRIVATE          = 0x00000002;

public static final int PROTECTED        = 0x00000004;

public static final int STATIC           = 0x00000008;
```
这样，如果需要需要判定它是不是 `public final static` ，则使用 `final static int PUBLIC_FINAL_STATIC = PUBLIC | FINAL | STATIC`。示例如下：
```java
public static boolean isPublicFinalStatic(int mod) {
  return (mod & PUBLIC_FINAL_STATIC) == PUBLIC_FINAL_STATIC;
}
```

## 3.4 简单的数据转换