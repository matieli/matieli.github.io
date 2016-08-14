---
layout: post
title: JVM调优细节
date: 2016-08-14
tag: JVM
---

* TOC
{:toc}

### **JVM调优-序**

> 1.Throughput GC指定方式： -XX:+UseParallelOldGC 或 -XX:+UseParallelGC，两者的区别是，-XX:+UseParallelOldGC同时激活新生代并行垃圾回收和老年代的并行垃圾回收，亦即，Minor GC和Full GC都是多线程的；-XX:+UseParallelGC只会激活新生代的并行垃圾回收。也就是使用了-XX:+UseParallelOldGC会自动激活-XX:+UseParallelGC。

> 2.在生产系统中，推荐打印出垃圾收集日志，开销很小，但能提供很多信息。推荐的命令行选项的最小集：-XX:+PrintGCTimestamps -XX:+PrintGCDetails -Xloggc:<fileName>-XX:+PrintGCDateStamps打印出的是日期，要求Java 6 Update 4及以上版本；-XX:+PrintGCTimeStamps打印出的是相对于JVM启动的时间偏移量。

> 3.老年代使用CMS（-XX:+UseConcMarkSweepGC），配合其使用的新生代垃圾收集器为多线程的ParNew；老年代使用Serial GC，配合其使用的新生代为单线程的DefNew。

> 4.为低等待时间（STW停顿）调优时，可以使用这两个选项:-XX:+PrintGCApplicationStoppedTime 、-XX:+PrintGCApplicationConcurrentTime

> 5.重视吞吐量和延迟的应用，应将-Xms和-Xmx设为一样，因为增加和缩小堆内存（包括老年代和新生代），需要Full GC。

> 6.几个内存设置选项：-XX:NewSize=<n>[g|m|k] —— 新生代的初始和最小内存。

>- 设置该选项时应该同时设置-XX:MaxNewSize=<n>[g|m|k].-XX:MaxNewSize=<n>[g|m|k] —— 新生代的最大内存。
>- 设置该选项时应该同时设置-XX:NewSize=<n>[g|m|k].-Xmn<n>[g|m|k] —— 将新生代的初始值、最小值、最大值设为同一个值。
>- 如果-Xmx和-Xms设的值不同，此时使用了-Xmn，则Java堆的增加和缩小都不会影响新生代的大小。因此，使用-Xmn的时候,-Xmx和-Xms的值应该是一致的。
>- -XX:PermSize=<n>[g|m|k] —— 永久代的初始和最小值。
>- -XX:MaxPermSize=<n>[g|m|k] —— 永久代的最大值。
>- -XX:MaxPermSize和-XX:PermSize设成一样，因为永久代的增加和缩小需要Full GC。

> 7.新生代的对象promote进老年代时空间不足，以及永久代空间不足会触发Full GC。

> 8.老年代空间不足触发了Full GC，不管永久代空间是否足够，都会对永久代进行回收；永久代不足触发的Full GC，即使老年代空间足，也会回收老年代。

> 9.老年代Full GC之前，会进行Minor GC，除非设置了-XX:-ScavengeBeforeFullGC。

> 10.可通过这种方式触发Full GC：jmap -histo:live <pid>，pid可以通过jps命令获得。

> 11.如果Minor GC太长，则应减小新生代的空间；若太频繁，则应增加新生代的空间。

> 12.调优改变新生代的大小时，尽量保持老年代的大小不变；调整老年代时，新生代保持不变。

### **计算LiveData Size**

>- Live Data Size是应用运行稳定，经历过Full GC之后老年代和永久代的堆内存占用。最好是观察多次取均值。
>- 强制Full GC：jmap –histo:live <pid>
>- <pid>可以通过jps命令获得。


### **初始堆大小**

> 一般，-Xms和-Xmx的初始值应当设为老年代Live Data Size的3到4倍；-XX:PermSize和-XX:MaxPermSize的初始值应当设为永久代Live Data Size的1.2到1.5倍；新生代的初始大小应当设为老年代Live Data Size的1到1.5倍。

### **为低延迟/响应性调优**

> 如果吞吐量GC的持续时间或频度比应用要求高很多的话，考虑换CMS。

### **细化新生代大小**

> 尽管Minor GC的时间与新生代中可触及的对象数有关，但一般来说，新生代越小，Minor GC耗时越短，但，减少新生代的大小，会增加Minor GC的频度。因为在相同的对象分配速率下，新生代越小，满的越快；相反，增加新生代的大小会降低Minor GC的频度。若发现Minor GC耗时太长，正确的做法是减小新生代的大小；若Minor GC太频繁，正确的做法是加大新生代的大小。

> 在改变新生代大小时，注意保持老年代不变。

> 根据计算，得到大概的对象分配速率。在增大新生代以减低频度的时候，计算出大概要增加到多大。

> 改变新生代大小时需要时刻注意：

>- 1.老年代的大小不应该小于Live Data Size的1.5倍；
>- 2.新生代的大小至少需要是堆大小（即，-Xmx和-Xms指定的）的10%；
>- 3.增加Java堆大小时，不要超过JVM可用的物理内存（超过可能导致操作系统使用虚拟内存）。

> 如果仍不能满足，只能要么改变应用，要么改变JVM部署模式，要么重新考虑应用的延迟要求。


### **细化老年代大小**

> 通过观察老年代空间增长以及每次Minor GC的时间戳得出晋升（promotion）速率。

> 计算出Full GC的大概频度，与应用需求相比，如果高于应用需求，增加老年代的大小以降低Full GC的频度，增加老年代大小时，注意保持新生代不变。

> 当修改老年代大小时，可能会导致应用只有Full GC，通常都是老年代空间不够大，即使Full GC后还是不足以放下所有从新生代晋升过来的对象导致的。

> 老年代空间不够的标识：Full GC后，新生代仍有大部分空间被占用。当老年代没有足够的空间来处理新生代晋升来的对象时，这些对象将回退（back up）到新生代。

### **为低延迟微调CMS**

> 若能避免CMS的内存整理操作，就可以避免Parallel GC那种STW整理，但空间不足时，仍会触发老年代的STW整理。

> CMS调优的目标：避免老年代的STW整理。

> 相比其它收集器，CMS需要更多微调，尤其是：

>- 1.微调新生代的大小；
>- 2.微调老年代并发收集的触发周期。

> 由于CMS收集老年代时大部分时候是与应用一起运行的，所以吞吐量会低一些，但延迟也会降低。

> CMS中，老年代空间耗尽，将会触发单线程的STW整理，这通常比吞吐量GC的Full GC时间长。

> 因此，避免老年代空间耗尽很重要！

> 从吞吐量GC迁移到CMS的一般原则是将老年代的大小增加20%到30%.

> CMS老年代碎片的解决办法：

>- 1.整理老年代（STW，应避免）；
>- 2.增加老年代大小（不能从本质上解决碎片问题，但能推迟因碎片而导致的STW整理的时间）；
>- 3.降低新生代对象晋升到老年代的频率。

> 与CMS GC不同，吞吐量GC默认启用了一个称为adaptive sizing的特性，会自动调整eden space和survivor space的大小。

> Minor GC时，如果to survivor space不够大，放不下所有从eden space和from survivor space拷贝过来的对象，放不下的部分将直接晋升到老年代，这可能导致老年代增长过快而触发STW整理，这是需要避免的事情。

> 避免survivor space不足而导致对象过早晋升到老年代的办法就是加大survivor space，改变survivor space大小的参数：

>- -XX:SurvivorRatio=<ratio> ratio必须大于0
>- Survivor space的大小 = -Xmn/(ratio+2) ，加2是因为有两个survivor space，ratio值越大，survivor space越小。

> 对于给定的新生代大小，减小survivor space ratio将增加survivor space的大小，同时减小eden space的大小；同样，增加survivor space ratio将减小survivor space的大小，同时增加eden space的大小。

> 需要意识到，减小eden space会增加Minor GC的频度，从而导致对象年龄上升过快。

> Tenuring threshold是一个对象的年龄，对象的年龄即该对象所经历的Minor GC的次数。当对象首次创建时，其年龄是0，经历一次Minor GC后，该对象仍然存活的话，其年龄为1.

> 新生代中，年龄超过JVM计算出的tenuring threshold的对象将会晋升到老年代。

> Tenuring threshold的计算是基于经历一次Minor GC后新生代中保存的可触及对象的大小以及target survivor space占用情况。相关的两个参数：

>- -XX:MaxTenuringThreshold=15
>- -XX:+PrintTenuringDistribution

### **Survivor space对象年龄**

> 通过-XX：+PrintTenuringDistribution打印出的信息，Desired Survivor Size=某个survivor space * target survivor ratio.

> Target survivor ratio是Hotspot用来表示survivor space目标占用百分比。

> 每个年龄以及该年龄所占的字节数：total是本行以及前面行所表示年龄占用的字节数之和。

> Desired survivor size小于total表示survivor space溢出，即，本次Minor GC中，部分对象直接晋升到了老年代。Survivor space溢出表示survivor space不足。

> 一般，new threshold 一直小于max tenuring threshold，或desired survivor size小于total survivor bytes表示survivor space小了。

> 加大survivor space会减小eden space，eden space减小会增加Minor GC的频度。如果这样导致了Minor GC不能满足应用要求，就要在增加survivor space的时候保持eden space不变。也就是，增加survivor space的时候，新生代也要增加。内存充足的情况下，尽量保持eden space不变。

> 如果发现从某个年龄开始往后的年龄的占用（total）空间没有什么变化，表示经历这么多次Minor GC，它们都没有得到回收。或许可以将MaxTenuringThreshold的值改小一些，以减小Minor GC时来回拷贝对象的开销。但是，如果从GC日志中没有发现MaxTenuringThreshold这个年龄所占用的空间，表示对象在到达最大年龄之前都被回收了。这样，一般将对象保留在新生代会更好，虽然有一些Minor GC拷贝的开销，但放到老年代回收则可能产生更多碎片以及可能触发整理老年代的操作（CMS GC）。

> -XX:TargetSurvivorRatio=<percent>.指定的值即survivor space的占用百分比，而不是ratio。模式是50.极少需要调整taeget survivor occupancy。

> CMS调优的目标是保持老年代的空闲空间，以避免STW整理。成功调优CMS要求老年代对象回收的速度至少达到新生代对象的晋升速度。要达到这个目标，可结合以下两点：

>- 1.足够大的老年代空间；
>- 2.尽早开始CMS收集，以使回收速度快于晋升速度。

> CMS周期的开始是基于老年代空间占用的。如果CMS周期启动的太迟，可能赶不上新生代对象晋升速度；如果启动的太早，会带来不必要的开销，还可能影响到应用的吞吐量。一般过早启动CMS周期比过晚启动好，因为晚起动的后果很糟糕。

> Hotspot试图自适应地计算出空间占用了多少就启动CMS周期。CMS中出现STW整理的关键词：concurrent mode failure。如果在GC日志中发现了这样的字样，可以使用下面的参数提早启动CMS周期：

>- -XX:CMSInitiatingOccupancyFraction=<percent>

> 指定的值表示老年代空间占用达到多少时会触发CMS周期的启动。与上面参数一起使用的一个参数是：

>- -XX:+UseCMSInitiatingOccupancyOnly

> 这个参数表示，总是使用-XX:CMSInitiatingOccupancyFraction所表示的比例来触发CMS周期的启动，否则，只会在第一次触发CMS周期的时候使用指定的比例，然后会自适应的调整后续CMS周期的启动时间。也就是说，首次CMS周期之后，就不再使用指定的比例了。

>- -XX:CMSInitiatingOccupancyFraction所指定的占用比例应该大于老年代Live Data Size所占的比例，否则,CMS收集器将不停地运行。-XX:CMSInitiatingOccupancyFraction的值至少需要是老年代Live Data Size的1.5倍。


### **CMS周期是否启动的太早或太迟**

> 太迟：CMS初始标记后不久就有Full GC。

> 太早：CMS周期中回收的空间很少，CMS初始标记到reset期间对空间占用下降的很少。

> CMS默认不回收永久代，尽管GC日志中有CMS Perm的字样。启动CMS回收永久代的参数：

>- -XX:+CMSClassUnloadingEnabled

> 如果是Java 6 Update 3或更早的版本，还需要启动以下参数：

>- -XX:+CMSPermGenSweepingEnabled

> 指定永久代占用多少时启动CMS回收：

>- -XX:CMSInitiatingPermOccupancyFraction=<percent>

>-（上面第一个参数也还是需要使用的，第二个参数看版本）

> 如果希望永久代一直按这个比例触发CMS回收，还必须指定：
>- -XX:+UseCMSInitiatingOccupancyOnly

> 一个CMS周期中有两个阶段是STW的：

>- 1.初始标记；
>- 2.重标记

> 尽管初始标记是单线程的，但很少会花费较长时间。重标记阶段是多线程的，可用的线程数可以通过下面的参数配置：

>- -XX:ParallelGCThreads=<n>

