# 在工作中使用并精通使用轮子，进而在学习中研究轮子

--- 

1. 将 `ArrayList` 转换为 `LinkedList` 用什么方法？各自的好坏？
2. 将集合类、数字做一次浅拷贝用什么方法？用 `for` 循环？

## 6.1 通过常量构造一个 `ArrayList` 返回。
源：
```java
List<String> list = new ArrayList();
list.add("a");
list.add("b");
list.add("c");
```
工具：
```java
List<String> list = Arrays.asList("a", "b", "c"); 
// 弊端：无法进行add、remove。可以使用new ArrayList(Arrays.asList("a", "b", "c"))包装一层以开发使用。
```

## 6.2 中文拼音排序
```java
Comparator c = java.text.Collator.getInstance(Locale.CHINA);
List<String> list = new ArrayList<>(Arrays.asList("张", "里", "s", "1"));
Collections.sort(list, c);
for (String s : list) {
    System.out.println(s);
}
String [] a = {"张", "里", "s", "1"};
Arrays.sort(a, c);
for (String s : a) {
    System.out.println(s);
}
```
关于排序不仅仅如此。 `Java` 提供了 `Comparaable` 和 `Comparator` 两种接口（两种接口时分开用的）， `Comparable` 接口需要让列表中的对象来实现接口中的方法 `compareTo(E)` ，返回正数表示当前对象比传入对象大，当前对象会排序靠后。如果想要按照多个字段排序，在这个方法内部也是可以完成的。  
  
大家可能发现这样的方式不是太灵活，因为一个对象实现接口后，这个方法就固定了。也就是说，它的排序算法已经固定了，如果它的排序算法不是固定的，是可以动态调整的，那么就用 `Comparator` 接口来扩展，它独立于被排序的对象单独存在，当需要排序的时候，以参数的形式传递，例如 `Locale.CHINA` ，你可以自己实现一个自定义对象的排序方式来满足特定对象的要求，只需要使用不同的 `Comparator` 实例即可。  
  
第三方工具包有很多，例如 `Guava`，[点击我查看详情](1)。但是，当遇到个性化问题的时候，并非别人没有提供现成的工具类自己就不写，个性化包装就是一个优秀程序员需要去承担的责任，所以老 A 程序员除了拥有非凡的功力外，也同时需要深知所合计领域的业务和个性化，这样才能针对业务做出一个“很爽”的架构体系。  
  
**根据实际的场景应当“因地制宜”才能“事半功倍”，同时要以大量的知识面为基础，充分利用现有的资源**。



[1]: http://www.cnblogs.com/peida/archive/2013/06/14/Guava_Optional.html
