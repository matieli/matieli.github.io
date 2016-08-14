---
layout: post
title: JVM介绍
date: 2016-08-14
tag: JVM
---

* TOC
{:toc}

### **序**

![image](\assets\images\jvm_introduce\1.png)

**引用计数算法**

给对象中添加引用计数器，每当有一个地方引用它时，计数器值+1；当引用失效时，计数器值-1；任何时候为0时就是不在使用。

>- 需要单独的字段存储计数器，增加了存储空间的开销；
>- 每次赋值都需要更新计数器，增加了时间开销；
>- 垃圾对象便于辨识，只要计数器为0，就可作为垃圾回收；
>- 及时回收垃圾，没有延迟性；
>- 不能解决循环引用的问题；

举例：对象objA和objB都有属性instance，赋值为objA. Instance=objB和objB. Instance=objA，除此之外这2个对象没有其他任何引用(已经可以确定为“垃圾”)，但他们因为相互引用着，导致他们的计数器都不为0，于是引用计数器无法通知GC回收他们。

```
/**
 * 引用计数
 * @author Tieli.Ma
 */
public class ReferenceCountingGC {
	public Object instance = null;
	private static final int _1MB = 1024 * 1024;
	/**
	 * 只是为了占点内存
	 */
	private byte[] bigSize = new byte[2 * _1MB];

	public static void main(String[] args) {
		ReferenceCountingGC objA = new ReferenceCountingGC();
		ReferenceCountingGC objB = new ReferenceCountingGC();
		objA.instance = objB;
		objB.instance = objA;

		objA = null;
		objB = null;

		// 假设在这里发生GC，那么objA与objB是否会被回收
		System.gc();

	}
}
```

引用计数算法(Reference Counting)的实现简单，判断也很高效，在大多数情况下是一个不错的算法。Java中没有使用它的主要原因是，它无法解决相互引用的问题。

**GC Roots**

可达性分析算法(根搜索算法)

通过一些列名为“GC Roots”的对象作为起始点，向下搜索，搜索经过的路径称为“引用链”。当一个对象找不到任何“连线”时，证明此对象是无用的。
几个对象之间相互有联系，但是GC Roots是不可达的，那么也是可回收的对象。

可作为GC Roots对象包含以下几种：

>- 虚拟机栈中引用的对象
>- 方法区中的类静态属性引用的对象
>- 方法区中的常量引用的对象
>- 本地方法中（native方法）引用的对象

**何谓引用**

Jdk1.2之后，引用分为强引用(Strong Reference)、软引用(Soft Reference)、弱引用(Weak Reference)、虚引用(Phantom Reference)，这4种引用强度依次减弱。

>- 强引用：Object o =new Object();只要强引用还在，则不会回收。
>- 软引用：描述非必须对象，内存溢出之前会再次回收，回收后还是不够，则抛出内存溢出。使用Jdk中SoftReference实现
>- 弱引用：描述非必须对象，回收时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。使用Jdk中的WeakReference实现。
>- 虚引用(幻影引用)：最弱的一种引用关系，不会对生存时间构成影响，也不会无法取得一个实例。用它的唯一目的，就是系统这个对象在被回收前，收到一个系统通知。使用Jdk中的PhantomReference实现

**救命稻草**

经过根搜索算法之后也并非是“非死不可”，一个对象的“死亡”必须经过2次标记过程；如果经过GC Roots之后没有引用链会进行第一次标记并进行一次筛选。筛选条件是是否有必要执行finalize();没有覆盖finalize()和已经被虚拟机调用过视为没必要执行。

如果有必要执行finalize()，扔到F-Queue队列，交给优先级很低的finalizer线程执行。执行含义为调用finalize()，但并不会等待方法结束；

稍后GC会对F-Queue中的对象小规模标记，如果说在finalize()中成功“解救自己”(重新与引用链上的任意对象关联，比如自己.this赋值给某类变量或对象成员变量)，那么在第二次标记的时就会被“解救”。如果在finalize()中未能逃脱，那么就“离死不远”了。

但这个“救命稻草”不要使用。finalize()只会被执行一次。

```
/**
 * Finalize()演示
 * 1.对象可以在被GC时自我拯救。 
 * 2.这种自救的机会只有一次，因为一个对象的finalize()方法最多只会被系统自动调用一次
 * @author Tieli.Ma
 */
public class FinalizeTest {
	private static FinalizeTest obj = null;
	public void isAlive() {
		System.out.println("yes, i am still alive :)");
	}
	@Override
	protected void finalize() throws Throwable {
		super.finalize();
		System.out.println("finalize mehtod executed!");
		FinalizeTest.obj = this;
	}
	public static void main(String[] args) throws Throwable {
		obj = new FinalizeTest();
		obj = null;// 对象第一次成功拯救自己
		System.gc();
		Thread.sleep(500);// 因为Finalizer方法优先级很低，暂停0.5秒，以等待它
		deadOrAlive();
		obj = null;// 这次自救却失败了
		System.gc();
		Thread.sleep(500);
		deadOrAlive();
	}

	private static void deadOrAlive() {
		if (obj != null) {
			obj.isAlive();
		} else {
			System.out.println("no, i am dead :(");
		}
	}

}
```

**回收方法区**

方法区也称永久代(JAVA8中使用元数据区Meta-space代替)

存储了每个类的信息（包括类的名称、方法信息、字段信息）、静态变量、常量以及编译器编译后的代码等。

主要回收废弃(没有任何地方引用)常量和无用的类

废弃常量包含字面量、类或接口、方法、字段的符号引用等

无用的类包含三个方面

>- 没有该类的任何实例存在
>- 加载该类的ClassLoader已经被回收
>- 该类对应的java.lang.Class对象没有任何地方引用，无法通过反射访问该类。

满足这三个条件时，虚拟机仅仅是“可以”回收，不像对象，不用了必然回收。可以通过虚拟机参数-Xnoclassgc控制。还可以使用 -verbose:class 查看加载的类。

大量使用反射、动态代理等频繁自定义ClassLoader场景需要虚拟机有类卸载机制，否则方法区可能会内存溢出。


### **垃圾收集算法**

#### **标记清除算法**

最基础的垃圾回收算法，后续算法都是基于它的思路和缺点而来

分为两个阶段“标记”和“清除”，先标记出所有要回收的对象，标记完成后统一回收。

> 缺点:

>- 效率：标记和清除2个过程的效率都不高
>- 空间：标记清除后会出现大量不连续的碎片，碎片太多可能导致某个对象来时，没有足够的联系的空间存储，而不得不在进行一次垃圾回收

![image](\assets\images\jvm_introduce\2.jpg)

#### **复制算法**

为了解决效率问题出现的复制算法

将可用内存划分成2块大小相等的块，每次使用其中一块。当这一块用完了就将还存活的对象复制到令一块。然后把使用过的内存清理掉。这样每次回收不会产生碎片只需移动堆的指针，实现简单、高效运行。

>缺点:

>- 每次可使用内存仅为一半。

![image](\assets\images\jvm_introduce\3.jpg)

#### **标记整理算法**

复制算法在对象存活较高时，执行较多复制操作，效率会降低

标记操作和“标记-清除”算法一致，后续操作不只是直接清理对象，而是在清理无用对象完成后让所有存活的对象都向一端移动，并更新引用其对象的指针。

> 缺点:

>- 在标记-清除的基础上还需进行对象的移动，成本相对较高，好处则是不会产生内存碎片。

![image](\assets\images\jvm_introduce\4.jpg)

#### **分代收集算法**

分代收集算法并不是一个新的算法

根据对象的存活周期的不同将内存划分为几块。一般把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。在新生代，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象存活率高、没有额外空间对他进行分配担保，就必须使用“标记-整理”和“标记-清除”算法进行回收。

看到这可能会有人问“复制算法不是只能使用分配的内存一半吗？为什么新生代还要使用它？”

因为新生代中分为3个区域，Eden、2个Survivor。Eden和其中一个Survivor一起工作，组成了复制算法中的一半，而另一个Survivor则是复制算法的另一半。JVM的Eden和Survivor默认比例为8：1，即Eden=80%，Survivor=10%，(有2个Survivor)，所以每次复制算法中“浪费”的资源只有10%。

### **垃圾收集器**

![image](\assets\images\jvm_introduce\5.jpg)

#### **概念理解**

并发和并行

>- 并行(Parallel)：指多条垃圾收集线程并行工作，但此时用户线程扔处于等待状态
>- 并发(Concurrent)：指用户线程和垃圾收集线程同时执行(但并不一定是并行的，可能为交替执行)，用户线程在继续执行，而垃圾收集器在另外一个cpu上执行

Minor GC 和 Major GC(Full GC)

>- 新生代GC(Minor GC)：指发生在新生代的垃圾收集动作，因为Java对象大多都具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快
>- 老年代GC（Major GC / Full GC）：指发生在老年代的GC，出现了Major GC，经常会伴随至少一次的Minor GC（但非绝对的）。Major GC的速度一般会比Minor GC慢10倍以上

STW：Stop The World，只进行垃圾回收，暂停一切用户线程

>- 目前为止的任何垃圾收集器都会stw，只是停顿的时间长短而已。

#### **串行GC**

Serial是基本的、历史悠久的、单线程执行、新生代收集器、使用复制算法、JVM Client模式下默认的新生代收集器。

对于限定单个cpu的环境来说，Serial由于没有线程交互的开销，专心做垃圾回收自然可以获得最高效的效率。

Serial Old是Serial收集器的老年代版本，它同样使用一个单线程执行收集，使用标记-整理算法。主要使用在Client模式下的虚拟机。


#### **并行GC**

Parallel Scavenge收集器也是一个新生代收集器，它也是使用复制算法的收集器，又是并行多线程收集器。Parallel Scavenge收集器的特点是它的关注点与其他收集器不同，CMS等收集器的关注点是尽可能地缩短垃圾收集时用户线程的停顿时间，而Parallel Scavenge收集器的目标则是达到一个可控制的吞吐量。

Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和标记-整理算法。

ParNew收集器其实就是Serial收集器的多线程版本，除了使用多条线程进行垃圾收集之外，其余行为与Serial收集器一样。

如果老年代使用CMS收集器，基本也只能和ParNew进行合作。


#### **并发GC-CMS**

CMS(Concurrent Mark Sweep)收集器是以保证最短回收停顿时间的收集器。基于标记清除算法实现的。收集过程分为以下：

>- 初始标记(STW initial mark）：初始标记仅仅只是标记一下GC Roots能直接关联到的对象，速度很快。
>- 并发标记(concurrent mark)：并发标记阶段就是进行GC Roots Tracing的过程。
>- 并发预处理(concurrent precleaning)：通过重新扫描，减少下一个阶段”重新标记”的工作。
>- 重新标记(STW remark)：重新标记阶段是为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录。
>- 并发清除(concurrent sweep)：并发清除阶段会清除对象。
>- 并发重置（concurrent reset）:重置cms的收集器，等待下次回收

![image](\assets\images\jvm_introduce\6.png)

优点：并发收集、低停顿。

缺点：

>- CMS收集器对CPU资源非常敏感。在并发阶段，虽然不会导致用户线程停顿，但是会占用CPU资源而导致引用程序变慢，总吞吐量下降。CMS默认启动的回收线程数是：(CPU数量+3) / 4。
>- CMS收集器无法处理浮动垃圾，在开始回收同时，程序在继续提供服务还是会产生可回收对象称为浮动垃圾。可能出现“Concurrent Mode Failure“，导致一次Full  GC。（出现“Concurrent Mode Failure“ 临时启用Serial Old收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了。）
>- CMS是基于“标记-清除”算法实现的收集器，使用“标记-清除”算法收集后，会产生大量碎片。

浮动垃圾，一般可通过设置尽可能早的触发GC而保证

碎片问题，CMS中有属性设置，GC后 伴随一次内存整理

#### **并发GC-G1**

G1(Garbage First)收集器是JDK1.7提供的一个新收集器，G1收集器基于标记-整理算法实现，也就是说不会产生内存碎片。还有一个特点之前的收集器进行收集的范围都是整个新生代或老年代，而G1将整个Java堆(包括新生代，老年代)。

G1特点

>- 并行和并发：利用多核、多cpu，少停顿
>- 分代收集：尽管不需要其他收集器配合，内部也是分代不同方式处理
>- 空间整合：标记—清理和复制两种算法结合
>- 可预测的停顿：可预测的时间停顿模型

G1收集器之所以能建立可预测的停顿时间模型，是因为它可以有计划地避免在整个Java堆中进行全区域的垃圾收集。G1跟踪各个Region里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region（这也就是Garbage-First名称的来由）。这种使用Region划分内存空间以及有优先级的区域回收方式，保证了G1收集器在有限的时间内可以获取尽可能高的收集效率。

G1执行过程

G1收集器的运作大致可划分为以下几个步骤：

>- 初始标记（Initial Marking）：仅仅只是标记一下GC Roots能直接关联到的对象，并且修改TAMS（Next Top at Mark Start）的值，让下一阶段用户程序并发运行时，能在正确可用的Region中创建新对象，这阶段需要停顿线程，但耗时很短。
>- 并发标记（Concurrent Marking）：从GC Root开始对堆中对象进行可达性分析，找出存活的对象，这阶段耗时较长，但可与用户程序并发执行。
>- 最终标记（Final Marking）：为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程Remembered Set Logs里面，最终标记阶段需要把Remembered Set Logs的数据合并到Remembered Set中，这阶段需要停顿线程，但是可并行执行。
>- 筛选回收（Live Data Counting and Evacuation）：首先对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间来制定回收计划，这个阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分Region，时间是用户可控制的，而且停顿用户线程将大幅提高收集效率。

#### **垃圾收集器总结**

![image](\assets\images\jvm_introduce\5.jpg)

如果垃圾收集算法是内存回收的方法论，垃圾收集器是具体实现。

上述对各个收集器进行了比较，但目的不是找出最好的，到现在为止都是个有利弊没有全能的。我们只能选择对具体应用选择最合适的收集器。

### **常见参数汇总**

1.堆设置

>- -Xms：初始堆大小
>- -Xmx：最大堆大小
>- -XX:NewSize=n:设置年轻代大小（-Xmn）
>- -XX:MaxNewSize=n:设置年轻代大小
>- -XX:NewRatio=n:设置年轻代和年老代的比值。默认：2。
>- -XX:SurvivorRatio=n:年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。默认：8，表示Eden：Survivor=8：2，一个Survivor区占整个年轻代的1/10
>- -XX:MaxTenuringThreshold：经历多少次Minor GC后到老年代
>- -XX:PretenureSizeThreshold：直接晋升到年老代对象的大小

2.收集器设置

>- -XX:+UseSerialGC：使用串行收集器
>- -XX:+UseParallelGC：使用并行收集器
>- -XX:+UseParalledlOldGC：使用并行年老代收集器
>- -XX:+UseConcMarkSweepGC：使用CMS+并发收集器

3.并行收集器设置

>- -XX:ParallelGCThreads=n:设置并行收集器收集时使用的CPU数。并行收集线程数。
>- -XX:MaxGCPauseMillis=n:设置并行收集最大暂停时间
>- -XX:GCTimeRatio=n:设置垃圾回收时间占程序运行时间的百分比。公式为1/(1+n)，默认为99

4.CMS收集器设置

>- -XX:UseCMSInitiatingOccupancyOnly:只有达到阈值时才CMS GC
>- -XX:CMSInitiatingOccupancyFraction: 堆上使用75％后开始CMS收集。
>- -XX:+UseCMSCompactAtFullCollection：在FULL GC的时， 对年老代的碎片整理
>- -XX:CMSFullGCsBeforeCompaction：执行多少次不压缩的Full GC后，执行一次带压缩的
>- -XX:+CMSScavengeBeforeRemark：强制remark之前开始一次minor gc
>- -XX:+CMSParallelRemarkEnabled：并行remark

5.辅助信息

>- -XX:+PrintGCDetails ：打印较为详细的GC信息
>- -XX:+PrintGC：打印GC信息
>- -XX:+PrintGCDateStamps：打印GC时间戳
>- -XX:+PrintClassHistogram ：打印类的直方图
>- -Xloggc:log/gc.log：打印GC信息到指定文件 可以设置绝对路径的。
>- -XX:+DisableExplicitGC：禁用system.gc
>- -XX:+ExplicitGCInvokesConcurrent：无论系统什么时候调用都执行CMS GC，而不是Full GC.
>- -XX:LargePageSizeInBytes：设置堆内存的内存页大小
>- -XX:-HeapDumpOnOutOfMemoryError：OOM时候生成dump
>- -XX:HeapDumpPath：xxx:dump文件路径

### **常见命令行工具**

1.jps --展示正在运行的虚拟机进程

```java
jps [Options] [hostid]
Options，选项，我们一般使用 -gcutil 查看gc情况hostid，VM的进程号，即当前运行的java进程号如：jps -l
展示当前机器正在运行的MainClass和jvmid
```

2.jinfo-- 实时查看和调整jvm参数

```java
Jinfo [Option] <Pid>
Options 选项
Pid  java进程
如 jinfo -sysprops 18500
查看18500的属性
如jinfo -flag LargePageSizeInBytes 17346
查看17346的LargePageSizeInBytes大小
```

3.jstat --JVM统计监测工具

```java
jstat [Options] vmid [interval] [count]
Options，选项，我们一般使用 -gcutil 查看gc情况vmid，VM的进程号，即当前运行的java进程号interval，间隔时间，单位为秒或者毫秒count，打印次数，如果缺省则打印无数次
如：jstat -gc 12538 5000
即会每5秒一次显示进程号为12538的java进成的GC情况
```

4.jmap--堆转储文件快照(dump文件)

```java
Jmap [Options] <Pid>
Options 选项
Pid  java进程
如 jmap –heap 12538
查看12538的堆内存
如jmap -dump:format=b,file=/home/app/dump1.bin 17346
生成dump文件
```

5.jhat --堆转储文件分析工具

```java
jhat [filename]
filename ，文件名
如：jhat /home/app/dump1.bin
分析dump文件，但尽量不用。原因1：耗资源；2：通过http方式可查看，但功能简陋
```

6.jstack--java堆栈跟踪工具

```java
Jstack [option] vmid
option:选项可通过-help查看
vmid :pid
如：jstack –l 19632
查看19632的线程堆栈信息
关于线程堆栈信息，java5之后再java.lang.Thread中增加了方法getAllStackTraces(),通过代码可以实现jstack的大部分功能。
```

### **常见可视化工具**

> Jconsole-Java监控与管理控制台

>- 内存、线程、类、VM等信息
>- 修改程序运行时的值



> VisualVM-多合一故障处理工具

>- 除了支持Jconsole所有功能外还支持性能分析。有大量插件可使用。

