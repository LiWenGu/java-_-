# 增加对原生态类型、集合类的说明

---

## 5.1 原生态类型
即不属于对象的那 5% 部分： `boolean` 、 `byte` 、 `short` 、 `char` 、 `int` 、 `float` 、 `long` 、 `double`。

与包装后的对象区别在于：包装后的对象会按照对象的规则存储在堆中，而“线程栈”上只存储引用地址。对象自然会占用相对较大的空间存放在堆中，在原生态类型中，“栈”上直接保存了它们的值，而不是引用（ `Reference` ）。  
 
例子如下：
```java
public static void main(String[] args) {
  Integer a = 1;
  Integer b = 1;
  Integer c = 200;
  Integer d = 200;
  println(a == b);  // true
  println(c == d);  // false
}
```
以下为Integer内源码：
```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];
    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```
内部维护了一个 `IntegerCache` 缓存，用于缓存 [-128, 127] 的数字（可以通过`-Djava.java.lang.Integer.IntegerCache.high=200` 设置high值），在编译阶段，将原始类型 `int` 赋值给 `Integer` 类型，会将原始类型自动编译为 `Integer.valueOf(int);` 即自动装箱。另外这种装箱操作是很隐藏的。例如，我们想用一个 `int` 类型的数字来作为 `HashMap` 的 `Key` ，那么在 `put()` 操作的时候就会自动装箱操作，再者 `List` 的 `add()` 操作。

### 5.1.1 横向扩展
1. `Boolean` 的两个值 `true` 和 `false` 都是 `cache` 在内存中的，自己 `new Boolean` 是另外一块空间。
2. `Byte` 的 256 个值全部 `cache` 在内存中，自己 `new Byte` 是另外一块空间。
3. `Short` 、 `Long` 两种类型的 `cache` 范围为 -128~127 ，无法调整范围，代码中写死，可以自己扩展。 
4. `Float` 、 `Double` 没有 `cache` ，可以自己扩展。

以下为测试代码：
```java
Byte a = Byte.valueOf((byte)130);
Byte b = Byte.valueOf((byte)130);
System.out.println(a == b);
```
以上可能涉及到原码、反码、补码的知识（-128~127的由来），[点我深入理解原码、反码、补码](1)

### 5.1.2 思维发散扩展
使用 `swich case` 或者使用 “>、<” 等比较操作时，会自动拆箱为原始类型进行比较，另外 `JDK1.7` 中 `swich case` 对 `String` 对象比较，其实是语法糖，编译后的字节码中依然是 `if else` 语句，并通过 `equals()` 实现的。

## 5.2 集合类
最简单的集合类为： `ArrayList` 、 `HashMap`。但是当你用 `ArrayList` 的时候是否想起了 `LinkedList` 、 `Vector` ；当你用 `HashMap` 的时候是否想起了 `TreeMap` 、 `HashSet` 、 `HashTable` ；当你要排序的时候是否想起了 `SortedSet` 等。如果使用 `ArrayList` 替换一个数据，我们会不会先 `remove` 再 `add` 一个数据，或是通过 `set(int index, E e)` 将对应下标的数据替换掉。下面的几个扩展，希望大家去思考。
1. 经常做修改操作的列表，是否考虑过使用 `LinkedList` ，因为 `ArrayList` 通常始终有些数组元素是空着的。
2. 在知道 `List` 长度范围的情况下，是否在实例化 `ArrayList` 的时候带上长度；这样降低了内存碎片和内存拷贝的次数。
3. 大家熟知的 `HashMap` 浪费空间更加严重，它的代码有一个 0.75 因子，当写入 `HashMap` 的数据个数（不是说所使用的数组下标个数，而是所有元素个数，也就是说，包含了同一个下标的链表中的所有元素个数）达到数组长度的 0.75 后，数组会自动扩展 1 倍，并且还需要做一个 `rehash` 操作，其实这个额时候也许很多桶上的节点都是空的。




[1]: http://www.cnblogs.com/zhangziqiu/archive/2011/03/30/ComputerCode.html



