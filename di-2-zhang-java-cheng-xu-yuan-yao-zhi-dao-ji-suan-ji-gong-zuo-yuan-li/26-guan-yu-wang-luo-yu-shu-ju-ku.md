# 啊，IO

---

## 6.1 `Java` 基本 `I/O`
`I/O` 的意思是输入/输出（ `Input/Output` ），只要是与内存程序通信的就可以将它理解为 `I/O` 。

1. 怎么理解 `CPU` 呢？ `CPU` 不是也与内存通信吗？
`CPU` 相对内存来讲级别更高，内存中的数据最终由 `OS` 调度给 `CPU` 来处理，也就是其它的设备要处理的数据通常会先放入内存中，然后再得到处理。而内存中处理好的数据反馈到某个设备上，这才算输入/输出。 `CPU` 是真正处理的实体，不算 `I/O` 设备。所有人要见 `CPU` 一般首先通过内存这一关。相对于程序内存，向其它的程序或设备写数据，称为“输出”，藏其它的程序或设备读取数据，称为“输入”。

## 6.2 `Java` 的网络基本原则
计算机 `I/O` 交互在网络传输过程是基于字节（ `byte` ）的，当然也可以说是二进制数据，不论你的程序是通过字符还是对象等方式，都会经过中间层处理转换为字节后发送到 `I/O` 端进行交互。而当传输中文字符比较特殊，因为计算机原本只有二进制 `byte` ，为了让人识别信息，才有了“字符”的概念，字符其实是计算机中使用数字表达的，我们看到的只是屏幕上最终的输出，作为程序员应当理解计算机内在是数字，也就是一连串的二进制数代表一个字符。  
  
对于任意一个字符（包含了汉字），使用什么数字来表达，有很多种方式，这就是“字符集”的概念，全世界有很多种字符，以 `ISO-8859-1` 为基础，最初只能表示 256 个字符，后来发展处各种各样的字符集，在中文界的代表为 `GB2312` 、 `GBK` 等，而 `UTF` 系列的通常是全球码，理论上支持世界上所有的字符，例如 `UTF-8` 就是大家熟知的字符编码， `Unicode` 也是大家熟知的字符集，还有 `UTF-16` 。  
  
接收方有没有办法自动知道发送方使用的字符集是什么？  
有两种方式：一是双方约定好编码；二是在数据的头部包装字符集。例如浏览器和服务器的交互使用头部的 `contentType` 参数作为传输数据类型、字符集的标识。  
  
接收方如何知道字节流结束了呢？  
一般是双方约定一种结束标记，或者在传输数据的头部指定位置标记 `body` 部分的字节长度。  

为何程序会经历那么多的拷贝才能做 `I/O` ？能否少拷贝几次？  
因为 `JVM` 的内存是自己管理的，是虚拟出来的，数据都在 `JVM` 中，要输出数据其实最终要靠 `OS` 和网络，理论上说必要的拷贝是必然的。而能否少做 `I/O` ，这在某些场景下的确可以做到（例如 `NIO` 中的 `DirectBuffer` ）。  
      
`Java` 的所有 `I/O` 都会经历一个 `Socket` 或 `Channel` 的过程，内在的基本道理差别不大，在输出数据时，这些数据一般会先经过 `JVM` ，然后到 `OS` 管理的一块内核态区域，再由 `Kernel` 去完成输出的交互，输出到网络或磁盘也类似这样。 
  
总的来讲，只要是基于 `Java` 开发的软件或工具，在基本功能和语法层面都要基于 `Java` 语言本身的机制，框架都是在 `Java` 语言本身提供的机制基础上做了很多包装的。如果要扩展底层的机制，则需要利用 `JNI` 来扩展（也需要 `JVM` 支持这种扩展）。例如， `Tomcat` 中的 `APR` 通信协议也是自己的 `JNI` 扩展，由于 `JNI` 需要更底层的 `C` 语言来编写，它需要考虑各种操作系统的情况，因此这样的扩展导致系统对本地的依赖性增强。

>用过不代表你的技术牛，没用过不代表没有能力或没有能力处理问题。其实有些时候不需要知道太多的使用细节，便知道它是如何实现的，因为你的技术功底和解决问题的能力在那里。技术时相通的，一切在于基本原理的变化，变化源于现实需求的抽象，才会演变出各种各样的技术。

## 6.3 `Java` 与数据库的交互
`Java` 与数据库通信在 `Socket` 基础上会建立许多不同的协议， `JDBC` 是其中一种，以它为例，数据库服务器端此时就类似于 `SocketServer` 提供 `Socket` 服务，内在提供配套的线程池来对应连接进行处理。 `Java` 本身通过 `JDBC` 的驱动程序与数据库端传送相应的指令来得到对应的结果（这个驱动程序内部就完成了通过 `Socket` 的协议包装）。  
  
这个过程看似简单，其实中间会有十分复杂的过程，客户端需要根据服务器端对应协议的要求来传送数据，这中间不再是我们所看到的 `SQL` 这样简单，所以需要数据库厂商将它封装为驱动。
>数据库服务器端的处理方式也很复杂，不同的数据库处理线程与资源分配的关系也是不同的，也有一些数据库是以进程为单位进行处理的，例如 `Oracle` 就是这样，并且会分配一个 `5MB` 左右的用户区域，那么自然的 `Oracle` 建立一个新连接的开销要大许多，但是用起来要顺很多；而 `MySQL` 并不是这样的。  
  
 `JDBC` 通常有 `Java` 语言标准接口，由第三方或数据库厂商来提供驱动程序的实现，也就是说，驱动程序就是实现接口的，主要组成的接口由 `Driver` 、 `Connection` 、 `DataSource` 、 `Statement` 、 `ResultSet` 等。操作数据库时最终都是通过与服务器端建立 `Socket` 来通信的。以 `MySQL` 为例，连接超时参数在 `MySQL` 的连接参数变成了 `connectTimeOut` ，每次网络操作发生读操作时使用的响应超时参数都为 `socketTimeOut` 。更多关于 `JDBC` 的源码在第八章详解。

