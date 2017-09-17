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
{
  public StringTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=3, args_size=1
         0: ldc           #2                  // String ab1
         2: astore_1
         3: ldc           #2                  // String ab1
         5: astore_2
         6: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         9: aload_1
        10: aload_2
        11: if_acmpne     18
        14: iconst_1
        15: goto          19
        18: iconst_0
        19: invokevirtual #4                  // Method java/io/PrintStream.println:(Z)V
        22: return
      LineNumberTable:
        line 3: 0
        line 4: 3
        line 5: 6
        line 6: 22
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 18
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream ]
        frame_type = 255 /* full_frame */
          offset_delta = 0
          locals = [ class "[Ljava/lang/String;", class java/lang/String, class java/lang/String ]
          stack = [ class java/io/PrintStream, int ]
}
SourceFile: "StringTest.java"
```