# 缓存可以理解为“就近原则”

---

## 5.1 缓存的相对性
1. 把代码这些**不怎么修改的数据**放在内存中访问会快很多的（将磁盘上的数据放在内存中，能得到快速的访问，这个我们可以认为是缓存）。
2. 网站中大部分流量来自 `JS` 、 `CSS` 等静态文件，希望在全国很多地方缓存这些信息，这样全国各地的朋友就近访问，那么效率自然就提升了（CDN技术：与“使用者”靠得更近，也叫缓存）。
3. 同一台电脑这个浏览器为什么比那个浏览器访问同一个网站要快很多么？（浏览器本身会缓存很多内容到本地，包括静态资源，DNS映射关系等）。
4. 将热点数据缓存在好的服务器上、分布式缓存或 `SSD` 硬盘上（更多是放在 `Redis` 中），在访问时自动路由到这上面。这也可以简单地称为缓存。
>接触更多的是，将数据写到磁盘上，中间一个 `Buffer` ，大小可控制，但这通常不会认为是 `Cache` ，只认为它是一个小数据作为缓冲。  
  
其实缓存无处不在：数据库的 `LRU` 队列以及 `Keep` 在内存中的存储过程也可以认为是缓存：使用 `ORM` 框架中类似的 `Hibernate` 和 `iBatis` 也有许多缓存机制： `CPU` 的三级缓存可以认为是缓存；操作系统的 `oscache` 也可以认为是缓存；在高端存储中为了提高磁盘访问效率也会使用缓存；在负载均衡设备中也经常会缓存部分静态页信息，用于直接输出，这也可以认为是缓存；将写少读多的数据放在类似的 `memcached` 中，这也是一个专门的分布式缓存。  
  
因此，说到缓存，并不一定非要联想到缓存设备，也不一定要联想到 `CPU` 的缓存。总之，不要将想法硬绑定到某种定义上，这是一个就近图快的概念，它以不同的形式来解决不同的问题。它可以被持久化（某个读取很快的硬盘中），也可以不被持久化（放在内存中（例如 `redis` ）或者 `CPU` 的三级缓存中），在实际的场景中只要访问的利用率能达到自己想要的最佳效果即可，而缓存本身只是一个相对的概念。

## 5.2 缓存的用途和场景
从作者的看法来讲，肯定要确定系统的缓存用途，例如要给应用系统构建缓存，做一些数据的缓存。在集群环境下，肯定优先选择分布式缓存。
>分布式缓存是一个共享的服务中心，一般不需要程序过于关心，程序员只需要通过 `API` 去访问它就可以了。这个服务中心可以是单机，也可以是集群，总之：提供数据的独立缓存服务。一般来说，要读取数据先到缓存中读取，如果读取不到再到数据库中读取，并将其放在缓存中，如果数据被修改了，那么缓存中对应的数据将失效或者被修改。  
  
这个看似简单的操作原理，其实在网络中实现起来可能会比较费劲，主要是因为要保持一致性。另外，读者朋友是否发现了一个问题：如果发起访问 10 次，有 5 次都在缓存中未命中，那么可能这 5 次操作既要访问缓存又要访问数据库：如果缓存中大量的写操作要与数据库同步，那么此时既要写缓存又要写入数据，这样有可能比直接访问数据库更慢。  
  
缓存用于“读多写少”的场景，数据中存在大量的读操作、少量的写操作，这是一个比例，通常读的比例比写的比例要大几倍、十多倍甚至几十倍，这时就会利用缓存的优势来降低数据库的压力（很多人都喜欢将用户登录后的基本信息放到缓存中，其实也就是一个用户在登录时发生一次写操作）。  
  
很多时候缓存用于“非强一致性”（在分布式环境下，由于网络延迟以及 `CAP` 理论，导致只能保证最终一致性），即使是现在的数据库，利用缓存的优势也是通过非绝对一致性完成的（数据库会通过日志、冗余等方法来完成丢失数据的回补工作）。宕机后有可能丢失少量数据是存在的，有些缓存是完全用内存实现的，自然更加严重，但通常我们认为宕机的概率很低，再加上这种机制可以带来巨大的性能优势，在某些场景下去做是值得的。  
  
通常缓存的访问效率很高，可以简单做一个对比：对于普通的 `MySQL` 数据库服务器，不论采用哪一种存储引擎，单机的 `QPS` 达到几千就很高了：但是对于缓存就完全不一样，可以达到几万甚至几十万以上，主要原因是缓存基于 `K-V` 结构，它剥离了数据库原本存在许多复杂的一致性概念，而且大部分情况下是在内存中完成的，通常它就处理简单的存取操作。  
  
总而言之，如果缓存的命中率可以达到 `95%` 以上，那么系统肯定算是比较成功地利用了缓存；反过来，如果缓存命中率很低，那么可以考虑放弃用缓存。  
  
换个角度讲，缓存让复杂性开始变强，让一致性不是那么容易保证，因此它对可用性的要求肯定没有数据库高，它存储的数据通常可以允许丢失一点点。  
  
在一个传统的应用系统中，访问量不大，非要用缓存，这没有错，但要看应用它是否足够简单。其实在绝大部分的传统应用中，数据库就足以支撑这些信息，只是很多时候大家自然地认为必须要用缓存。