# 讲一些运算符，然后再讲讲“大数字”是如何处理的。

---

## 3.1 变量 A 、 B 交换有几种方式
1. 使用中间量。
2. 使用数据叠加后减回来：
```java
A = A + B;
B = A - B;
A = A - B;
```
3. 使用异或：
```java
A = A ^ B;
B = A ^ B;
A = A ^ B;
```

## 3.2 将无序数据 Hash 到指定的板块
假设有一个长度为 5000 的数组，希望写入的数据能按照某种规则读取出来，因此不能随机写入，那么数据应当与数组的下标产生某种关系，这样写入和读取只要按照相同的规则就可以达到目的了。即数据 % 5000 就可以得到数组的下标了。当然也可以使用 数据 & 4999 来得到下标，但是会有一个问题：假设数组长度为 11 ，则使用 10 来进行 & 操作。那么， 0001 、 0100 、 0101 这些数字永远都会放在下标为 0 的地方，而导致没有数据。

## 3.3 大量判定“是|否”的操作
以下为java.lang.reflect.Modifier类的源码：
```java
public static final int PUBLIC           = 0x00000001;
 
public static final int PRIVATE          = 0x00000002;

public static final int PROTECTED        = 0x00000004;

public static final int STATIC           = 0x00000008;

public static final int FINAL            = 0x00000010;
```
这样，Java程序在读取字节码时，读取到一个数字，该数字只需要与对应的编码进行 & 操作，genuine结果是否为 0 即可得到答案。而且还可以同时判断是否属于多个，如下代码：
```java
final static int PUBLIC_FINAL_STATIC = PUBLIC | FINAL | STATIC 
public static boolean isPublicFinalStaticc(int mode) {
    return (mod & PUBLIC_FINAL_STATIC) == PUBLIC_FINAL_STATIC;
}
```
## 3.4 简单的数据转换
1. 将十进制数据转换为“二进制的字符串”，使用 `Integer.toBinaryString(int x)` 。数据存储本身以二进制存储，这只是转化成二进制用于UI的展示。
2. 将十进制数据转换为“十六进制的字符串”，使用 `Integer.toHexString(int x)` 。
3. 其他进制的数据转换为十进制数据，使用 `Integer.parseInt(String, int)` 、 `Integer.valueOf(String, int)` 。其中后者在源码上直接调用的前者。  
  
数字与 `byte[]` 之间的转换是什么意思呢？大家都知道，在 `Java` 语言中， `int` 是由 4 个字节（ `byte` ）组成的。在网络上发送数据的时候，都是通过 `byte` 流来处理的，所以会发送 4 个 `byte` 的内容， 4 个 `byte` 会由高到低顺序排列发送，接收方反向解析。在 `Java` 中可以基于 `DataOutputStream` 、 `DataInputStream` 的 `writeInt(int)` 和 `readInt()` 来得到正确的数字。源码如下：
```java
public final void writeInt(int v) throws IOException {
    out.write((v >>> 24) & 0xFF);
    out.write((v >>> 16) & 0xFF);
    out.write((v >>>  8) & 0xFF);
    out.write((v >>>  0) & 0xFF);
    incCount(4);
}
```
```java
public final int readInt() throws IOException {
    int ch1 = in.read();
    int ch2 = in.read();
    int ch3 = in.read();
    int ch4 = in.read();
    if ((ch1 | ch2 | ch3 | ch4) < 0)
        throw new EOFException();
    return ((ch1 << 24) + (ch2 << 16) + (ch3 << 8) + (ch4 << 0));
}
```

## 3.5 数字太大， `long` 都存放不下
`long` 采用 8 个字节来存放数字，如果遇到了变态的设置，需要使用 `BigDecimal` 来操作，它不是以固定长度的字节来存储数据，而是以对象的方式管理位的，支持加减乘除，由于是对象，因此需要调用对应的方法来实现加减乘除，其中除的方法需要指明小说点个数以及是否四舍五入等。