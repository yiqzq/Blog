HashMap,HashTable,ConcurrentHashMap三者的联系与区别(有坑待填

[toc]

关于HashMap,前面已经研究过了[HashMap的相关问题](https://blog.csdn.net/yiqzq/article/details/104800744)

但是HashMap是线程不安全的,所以接下来研究一下线程安全的HashTable,ConcurrentHashMap与HashMap的区别

## 区别

1. HashMap允许插入null键和null值，但是HashTable和ConcurrentHashMap是不允许插入null键和null值的。
2. 扩容机制：
   - HashTable 初始size为**11**，扩容：newsize = olesize*2+1
   - HashMap 初始size为**16**，扩容：newsize = oldsize*2

3. 底层实现：

HashMap采用的是数组+链表+红黑树

HashTable采用的是数组+链表

ConcurrentHashMap采用分段的数组+链表实现

4. 同步的实现：

   HashTable对每一个会修改集合的方法添加了synchronized，也就是每一次修改都是锁住了整个对象，效率很低，基本被淘汰了，在JDK1.5，使用ConcurrentHashMap实现多线程下同步的功能。

## 一些面试问题

### 如何实现HashMap的线程安全

- 使用Collections.synchronizedMap(Map)创建线程安全的map集合
- Hashtable
- ConcurrentHashMap

#### Collections.synchronizedMap(Map)的底层实现

```java
private static class SynchronizedMap<K,V>
        implements Map<K,V>, Serializable {
        private static final long serialVersionUID = 1978198479659022715L;

        private final Map<K,V> m;     // Backing Map
        final Object      mutex;        // Object on which to synchronize

        SynchronizedMap(Map<K,V> m) {
            this.m = Objects.requireNonNull(m);
            mutex = this;
        }

        SynchronizedMap(Map<K,V> m, Object mutex) {
            this.m = m;
            this.mutex = mutex;
        }
    /*
    	public int size() {
            synchronized (mutex) {return m.size();}
        }
        public boolean isEmpty() {
            synchronized (mutex) {return m.isEmpty();}
        }
        ...省略代码
        */
}
```

可以看到，在代码添加了互斥变量mutex，同时也提供两种构造器，一种是默认的mutex，一种允许传入自定义mutex。然后在这之后的代码都都会对添加synchronized代码块以保证同步，也说明并发性能也不是很好。

### 快速失败机制和安全失败机制

在使用迭代器对集合对象进行遍历的时候，如果 A 线程正在对集合进行遍历，此时 B 线程对集合进行修改（增加、删除、修改），那么就会抛出ConcurrentModificationException 异常，这称为快速失败机制。

在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历，这种操作称为安全失败机制。

### Synchronized锁升级

![img](https://pic1.zhimg.com/80/v2-e47232518a4e042f31a9e0eb6a48f88c_720w.jpg)

首先需要知道一个东西，Java的对象头,由Mark World ，指向类的指针，以及数组长度三部分组成,存储在JVM堆中,用于保存对象的信息。其中有个叫Mark World 的，记录了对象和锁有关的信息，上图是Mark World 的结构。

可以看到有4种锁，按照锁升级的顺序排序，**无锁->偏向锁 -> 轻量级锁 -> 重量级锁**，消耗也是从左到右增加。

#### 偏向锁的引入

经过HotSpot的作者大量的研究发现，大多数时候是不存在锁竞争的，常常是一个线程多次获得同一个锁，因此如果每次都要竞争锁会增大很多没有必要付出的代价，为了降低获取锁的代价，才引入的偏向锁。

#### 偏向锁的使用

这个流程网上看到了不同版本，也不知道哪个是真实的（雾），另外一个版本是说如果线程ID不同，通过CAS操作去获得偏向锁，如果失败，就会升级锁（可能就是下面2步中的简化吧）

- 当一个线程获得一个对象锁的时候，如果是无锁状态，那就设置偏向状态，将当前进程ID记录在Mark World中。

- 如果已经设置了偏向状态，查看Mark World中的线程ID是不是和当前线程ID一样
  - 如果一样，说明是同一个线程，直接访问
  - 如果不一样，说明不是一个线程，那就查看Mark World中的的标记线程是否还存活
    - 如果不存活，锁对象就被重置为无锁状态，然后当前线程可以获得锁，同样设置偏向状态和线程ID
    - 如果存活，立刻查找存储线程的栈帧信息
      - =如果还是需要继续持有这个锁对象，那就暂停线程，升级为轻量级锁
      - 如果不需要，将锁对象设位无锁，设置为偏向锁

**需要注意的几点**

1. 偏向锁不会主动释放锁，因为偏向锁主要是依靠线程ID相不相同去判断的。

2. 偏向锁的撤销需要等待全局安全点



#### 轻量级锁的引入

轻量级锁考虑的是竞争锁对象的线程不多，而且线程持有锁的时间也不长的情景。因为阻塞线程需要CPU从用户态转到内核态，代价较大，如果刚刚阻塞不久这个锁就被释放了，那这个代价就有点得不偿失了，因此这个时候就干脆不阻塞这个线程，让它自旋这等待锁释放。

#### 轻量级锁的使用

1. 线程由偏向锁升级为轻量级锁时，会先把锁的对象头MarkWord复制一份到线程的栈帧中，建立一个名为锁记录空间（Lock Record），用于存储当前Mark Word的拷贝。
2. 虚拟机使用cas操作尝试将对象的Mark Word指向Lock Record的指针，并将Lock record里的owner指针指对象的Mark Word。
3. 如果cas操作成功，则该线程拥有了对象的轻量级锁。第二个线程cas自选锁等待锁线程释放锁。
4. 如果多个线程竞争锁，轻量级锁要膨胀为重量级锁，Mark Word中存储的就是指向重量级锁（互斥量）的指针。其他等待线程进入阻塞状态。

#### 重量级锁

自旋失败，很大概率 再一次自选也是失败，因此直接升级成重量级锁，进行线程阻塞，减少cpu消耗。

当锁升级为重量级锁后，未抢到锁的线程都会被阻塞，进入阻塞队列。

### ConcurrentHashMap的jdk1.7实现

![img](https://user-gold-cdn.xitu.io/2019/12/17/16f140880441eab3?imageslim)

如图所示，底层是由 Segment 数组、HashEntry 组成，和 HashMap1.7 一样，仍然是**数组加链表**。

#### 能够并发的原因

ConcurrentHashMap 采用了**分段锁**技术，其中 Segment 继承于 ReentrantLock。

不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理，理论上 ConcurrentHashMap 支持 CurrencyLevel (Segment 数组数量)的线程并发。

每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment。

#### put操作

1. 通过key定位到对应的Segment
2. 尝试自旋获取锁
3. 当重试的次数达到了 `MAX_SCAN_RETRIES` 则改为阻塞锁获取，保证能获取成功。

#### get操作

 1.Key 通过 Hash 之后定位到具体的 Segment 

2. 再通过一次 Hash 定位到具体的元素上
3. 获得值

### ConcurrentHashMap的jdk1.8实现

#### 能够并发的原因

采用了 `CAS + synchronized` 来保证并发安全性。

#### put操作

1. 通过key计算hashcode
2. 判断是否需要初始化
3. 为当前 key 定位出的 Node
   1. 如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
   2. 如果当前位置的 `hashcode == MOVED == -1`,则需要进行扩容。
   3. 如果都不满足，则利用 synchronized 锁写入数据。
      1. 如果数量大于 `TREEIFY_THRESHOLD` 则要转换为红黑树。

#### get操作

1. 两次hash计算出hash值
2. 检查头节点是否匹配
3. 如果不行就检查红黑树
4. 还是不行就链表检查

#### 扩容机制//待补,很高深的样子

## 参考资料

[《我们一起进大厂》系列-ConcurrentHashMap & Hashtable](https://juejin.im/post/5df8d7346fb9a015ff64eaf9#heading-1)

<https://www.jianshu.com/p/e5ee7beadb45>