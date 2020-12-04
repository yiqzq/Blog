# 深入理解Java虚拟机读书笔记(三）

[toc]

## 对象分配的策略

1. **对象优先在Eden分配**

2. **大对象直接进入老年代**

   -XX：PretenureSizeThreshold : 指定大于该设置值的对象直接在老年代分配

3. **长期存活的对象将进入老年代**

   -XX： MaxTenuringThreshold : 对象晋升老年代的年龄阈值

4. **动态对象年龄判定**

   如果在Survivor空间中相同年龄所有对象大小的总和大于 Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代

5. **空间分配担保**

   在发生Minor GC之前，虚拟机必须先检查老年代最大可用的连续空间是否大于新生代所有对象总
   空间，如果这个条件成立，那这一次Minor GC可以确保是安全的。如果不成立，则虚拟机会先查看XX：HandlePromotionFailure参数的设置值是否允许担保失败（Handle Promotion Failure）；如果允 许，那会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大 于，将尝试进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于，或者-XX： HandlePromotionFailure设置不允许冒险，那这时就要改为进行一次Full GC。

## 故障检测工具

**jps：虚拟机进程状况工具**

`jps [-q] [-mlvV] [<hostid>]`

**jstat：虚拟机统计信息监视工具**

`jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]`

<img src="https://i.loli.net/2020/04/09/PStcxqbiRIjwLrd.png" alt="image-20200409121259831" style="zoom:80%;" />

解释一下各种参数缩写

有一个官方文档链接:https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html

S0,S1：表示两块Survivor区

E：Eden区

O:：Old,老年代

M：metaspace,元空间(在jdk7及以下应该是P,表示永久代)
CCS：压缩类(不知道是啥)

YGC：Young GC,年轻代GC次数

YGCT：Young GC TIme，年轻代GC总时间

FGC：Full GC次数

FGCT：Full GC总时间

GCT：GC Time,GC 总时间

**jinfo：Java配置信息工具**

实时查看和调整虚拟机各项参数

`jinfo [option] <pid>`

**jmap：Java内存映像工具**

jmap（Memory Map for Java）命令用于生成堆转储快照

**jhat：虚拟机堆转储快照分析工具**

**jstack：Java堆栈跟踪工具**

## 可视化故障处理工具

JConsole，JHSDB，VisualVM，JMC

**JHSDB：基于服务性代理的调试工具**

**JConsole（Java Monitoring and Management Console）是一款基于JMX（Java Manage-ment**
**Extensions）的可视化监视、管理工具**

**VisualVM（All-in-One Java Troubleshooting Tool）是功能最强大的运行监视和故障处理程序之一**

**Java Mission Control：可持续在线的监控工具**

## JVM调优实例

**大内存硬件上的程序部署策略**

对于堆内存很大，如果采用的是高吞吐量的垃圾收集器，那么必然导致每一次垃圾收集的停留时间会很长，一种折中策略就是调小堆内存，增加GC次数，采用低延迟的垃圾收集器，通过降低吞吐量来获得更好的用户体验。但是调低堆内存的策略如果服务器的性能较好，那就会显得很浪费硬件资源。

**目前单体应用在较大内存的硬件上主要的 部署方式有两种：**
**1）通过一个单独的Java虚拟机实例来管理大量的Java堆内存。**

采用这个方法主要是要控制Full GC的次数，尽量可以在深夜运行

1. 回收大块堆内存而导致的长时间停顿
2. 大内存必须有64位Java虚拟机的支持，但由于压缩指针、处理器缓存行容量（Cache Line）等因
   素，64位虚拟机的性能测试结果普遍略低于相同版本的32位虚拟机。
3. 必须保证应用程序足够稳定，因为这种大型单体应用要是发生了堆内存溢出，几乎无法产生堆转
   储快照。

 **2）同时使用若干个Java虚拟机，建立逻辑集群来利用硬件资源。**

	1. 节点竞争全局的资源，最典型的就是磁盘竞争
 	2. 很难最高效率地利用某些资源池，譬如连接池，
 	3. 如果使用32位Java虚拟机作为集群节点的话，各个节点仍然不可避免地受到32位的内存限制，在
     32位Windows平台中每个进程只能使用2GB的内存，考虑到堆以外的内存开销，堆最多一般只能开到 1.5GB。
 	4. 大量使用本地缓存（如大量使用HashMap作为K/V缓存）的应用，在逻辑集群中会造成较大的内
     存浪费，因为每个逻辑节点上都有一份缓存，这时候可以考虑把本地缓存改为集中式缓存。

**集群间同步导致的内存溢出**

**堆外内存导致的溢出错误**

一台32位的计算机，由于最大内存限制最多为4GB,所以对一个进程如果分配比较多的堆内存同时系统中存在堆外内存的分配（比如NIO中的DirectByteBuffer）虚拟机虽然会对直接内存进行回收，但是直接内存却不能像新生代、老年代那样，发现空间不足了就主 动通知收集器进行垃圾回收，它只能等待老年代满后Full GC出现后，“顺便”帮它清理掉内存的废弃对象。

**除了堆和方法区之外，其余也需要内存的地方**

**直接内存**：可通过-XX：MaxDirectMemorySize调整大小，内存不足时抛出OutOf-MemoryError或
者OutOfMemoryError：Direct buffer memory。

 **线程堆栈**：可通过-Xss调整大小，内存不足时抛出StackOverflowError（如果线程请求的栈深度大于虚拟机所允许的深度）或者OutOfMemoryError（如果Java虚拟机栈容量可以动态扩展，当栈扩展时 无法申请到足够的内存）。

**Socket缓存区**：每个Socket连接都Receive和Send两个缓存区，分别占大约37KB和25KB内存，连接
多的话这块内存占用也比较可观。如果无法分配，可能会抛出IOException：Too many open files异常。

 **JNI代码**：如果代码中使用了JNI调用本地库，那本地库使用的内存也不在堆中，而是占用Java虚拟机的本地方法栈和本地内存的。 

**虚拟机和垃圾收集器**：虚拟机、垃圾收集器的工作也是要消耗一定数量的内存的。

**外部命令导致系统缓慢**

**服务器虚拟机进程崩溃**

异步调用太多导致的崩溃

**不恰当数据结构导致内存占用过大**

在HashMap<Long，Long>结构中，只有Key和Value所存放的两个长整型数据是有效数据，共16字节（2×8字节）。这两个长整型数据包装成java.lang.Long对象之 后，就分别具有8字节的Mark Word、8字节的Klass指针，再加8字节存储数据的long值。然后这2个 Long对象组成Map.Entry之后，又多了16字节的对象头，然后一个8字节的next字段和4字节的int型的 hash字段，为了对齐，还必须添加4字节的空白填充，最后还有HashMap中对这个Entry的8字节的引 用，这样增加两个长整型数字，实际耗费的内存为(Long(24byte)×2)+Entry(32byte)+HashMap Ref(8byte)=88byte，空间效率为有效数据除以全部内存空间，即16字节/88字节=18%，这确实太低了。

**由Windows虚拟内存导致的长时间停顿**

怀疑GUI程序在最小化时它的工作内存被 自动交换到磁盘的页面文件之中了，这样发生垃圾收集时就有可能因为恢复页面文件的操作导致不正 常的垃圾收集停顿。

## class文件

**class文件结构**

1. 每个Class文件的头4个字节被称为**魔数**（Magic Number），它的唯一作用是确定这个文件是否为
   一个能被虚拟机接受的Class文件。

2. 紧接着魔数的4个字节存储的是Class文件的**版本号**：第5和第6个字节是次版本号（Minor
   Version），第7和第8个字节是主版本号（Major Version）

3. 再然后就是**常量池**了。

4. 常量池结束之后，紧接着的2个字节代表**访问标志**

5. 再之后就是**类索引、父类索引与接口索引集合**，类索引（this_class）和父类索引（super_class）都是一个u2类型的数据，而接口索引集合（interfaces）是一组u2类型的数据的集合，Class文件中由这三项数据来确定该类型的继承关系。

6. 再之后是**字段表**（field_info）用于描述接口或者类中声明的变量。Java语言中的“字段”（Field）包括类级变量以及实例级变量，但不包括在方法内部声明的局部变量

7. 之后就是**方法表**

   \<init>和\<clinit>的区别，init是对象构造器调用的，clinit是类构造器，也就是初始化静态变量的。

   **关于方法签名：**

   字节码层 面的方法特征签名以及Java代码层面的方法特征签名，Java代码的方法特征签名只包括方法名称、参数 顺序及参数类型，而字节码的特征签名还包括方法返回值以及受查异常表

8. **属性表**，比如说方法体内的代码就保存在Code中属于这个属性表

**常量池**

常量池中主要存放两大类常量：字面量（Literal）和符号引用（Symbolic References）。

**字面量**：文本字符串、被声明为final的常量值

**符号引用**：

被模块导出或者开放的包（Package） 

类和接口的全限定名（Fully Qualified Name） 

字段的名称和描述符（Descriptor） 

方法的名称和描述符 

方法句柄和方法类型（Method Handle、Method Type、Invoke Dynamic） 

动态调用点和动态常量（Dynamically-Computed Call Site、Dynamically-Computed Constant）

**Java采用的是动态连接的方式。**

## 字节码指令

**加载和存储指令**

**load**，将一个局部变量加载到操作栈

**store**，将一个局部变量加载到操作栈

**const**， 将一个常量加载到操作数栈

**运算指令**

加法指令：iadd、ladd、fadd、dadd 

减法指令：isub、lsub、fsub、dsub 

乘法指令：imul、lmul、fmul、dmul 

除法指令：idiv、ldiv、fdiv、ddiv 

求余指令：irem、lrem、frem、drem 

取反指令：ineg、lneg、fneg、dneg 

位移指令：ishl、ishr、iushr、lshl、lshr、lushr 

按位或指令：ior、lor 

按位与指令：iand、land 

按位异或指令：ixor、lxor 

局部变量自增指令：iinc 

比较指令：dcmpg、dcmpl、fcmpg、fcmpl、lcmp

**类型转换指令**

i2b、i2c、i2s、l2i、f2i、f2l、d2i、d2l和d2f

**对象创建与访问指令**

**操作数栈管理指令**

**控制转移指令**

**方法调用和返回指令**

**异常处理指令**

**同步指令**