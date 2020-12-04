# AQS 学习

[toc]

`AQS `就是`AbstractQueuedSynchronizer`,抽象的队列同步器，AQS实现了对**同步状态的管理，以及对阻塞线程进行排队，等待通知**等等一些底层的实现处理。AQS的核心也包括了这些方面:**同步队列，独占式锁的获取和释放，共享锁的获取和释放以及可中断锁，超时等待锁获取这些特性的实现**

**前置知识**

```java
//这是AQS的三个很重的变量，也可以说明等待队列底层其实是一个链表  
//头节点，不保存真实信息，只表示位置，不代表实际的等待线程
private transient volatile Node head;
//尾节点
private transient volatile Node tail;
//状态
private volatile int state;
```

可以看一下这个图，加深对head和tail的理解

![img](https://user-gold-cdn.xitu.io/2018/11/27/16754303119073a8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**FairSync的继承结构，它其实是间接继承自AQS的**

![image-20200319163345341](https://i.loli.net/2020/03/19/as8bM1FnEqlytBx.png)

## lock加锁过程

**这里以FairSync(公平锁)的lock方法为例，解析一下lock过程，以及AQS中的一些基础方法**

### acquire方法

```java
//FairSync的lock方法
final void lock() {
    acquire(1);
}
/*
AQS的accquire方法
1. 会先调用tryAcquire方法,这个方法是在AQS中并没有用具体的实现，只抛出了一个异常，在FairSync中重写了这方法，
2. tryAcquire会先尝试获得锁，如果返回true，也就是获得锁成功，导致逻辑短路，那么结束了
3. 如果tryAcquire返回false，也就是获得锁失败，就要往阻塞队列添加当前线程，这是addWaiter方法实现的功能，具体参考下方源码分析
4. acquireQueued方法可以对排队中的线程进行“获锁”操作。
*/

 public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            //如果返回打断,就要执行下面的代码
            //也就是说	获取了锁以后还要中断线程
            selfInterrupt();
    }

```

### tryAcquire方法

```java
/*
这个方法的作用是尝试获得锁，如果获得成功就返回true，失败则返回false
*/
protected final boolean tryAcquire(int acquires) {
    		//获得当前线程	
            final Thread current = Thread.currentThread();
    		//state是AQS一个很重要的变量，为0表示锁还没有被获取
            int c = getState();
            if (c == 0) {
                //如果返回了false， 就说明可以尝试获得锁
                if (!hasQueuedPredecessors() &&
                    //compareAndSetState方法就是以CAS的方式设置state，保证原子性
                    compareAndSetState(0, acquires)) {
                    //设置当前拥有独占访问的线程，这个方法内部就一句话，应该不需要解释了吧
                    //clusiveOwnerThread = thread;
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
    		//如果c==1,就是锁已被获得，那就查看获得锁的线程是不是当前线程
            else if (current == getExclusiveOwnerThread()) {
                //一般就是+1,可重入锁的表现
                int nextc = c + acquires;
                //int溢出为负数，也就是超过最大锁数
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                //那就增加可重入锁的参数
                //另外说一句，这边为什么没有采用CAS的方式设置state，我觉得是因为这里灭有并发差生的问					//因为最多只有一个线程能够进入到这个代码块
                setState(nextc);
				//返回true表示获得锁成功
                return true;
            }
    		//要么就是c=0，但是自旋设置锁，要么就是锁已被占用
            return false;
        }
```



### hasQueuedPredecessors方法

```java
/*
判断阻塞队列是否有等待的线程，如果有返回true，没有返回false
1. 如果h==t 说明还没有初始化,h和t都是null,或者是初始化了,但是队列中还没有元素
2. (s = h.next) == null,这个判断是为了在并发情况下可能会产生的问题,详细问题可以看下面的enq方法
3. s.thread != Thread.currentThread()说明阻塞队列的第一个有效节点线程与当前线程不同，当前线程必须加入进等待队列，也就是有比当前线程优先级高的线程
*/
public final boolean hasQueuedPredecessors() {
    Node t = tail; // Read fields in reverse initialization order
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

### enq方法

```java
/*
入队操作并不是一个原子操作,所以就会存在初始化完成,也就是进入else代码块,将node节点设置为tail,使得head!=tail,但是t.next=node并没有执行,也就是导致hasQueuedPredecessors中(s = h.next) == null的问题,这是需要判断出阻塞队列中有元素的,只是因为在并发环境下,设置了tail指向head节点的信息,但没有设置head指向tail指针。
这段话解释hasQueuedPredecessors方法中的问题
——————————————————————————————————————————————————————————————————————————————————
我们由下面的分析可以知道，进入enq有两个可能一个是没有初始化，一个是CAS竞争失败，都会在这个得到体现
*/
private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
         	//没有初始化
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } 
            /*进到这里有两种可能
            1. 没有初始化，那么初始化完成之后，会同时添加node到阻塞队列
            2. CAS竞争失败进入到这里，那么就会再次尝试CAS的方式添加到队尾，注意这是个死循环，只有当自旋				成功的时候，才会return
            */
            //addWaiter一样的操作，不过是变量名换了一下
            else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```



### compareAndSetTail方法

就是CAS的实现。

compareAndSetTail方法，完成尾节点的设置。这个方法主要是对tailOffset和Expect进行比较，如果tailOffset的Node和Expect的Node地址是相同的，那么设置Tail的值为Update的值。

### addWaiter方法

```JAVA
/*
1. 传入的参数mode是Node.EXCLUSIVE，是AQS中Node内部类的一个常量
 static final Node EXCLUSIVE = null;

*/
private Node addWaiter(Node mode) {
 		   //	获得当前线程封装成Node
        Node node = new Node(Thread.currentThread(), mode);
        // 获得tail尾节点，尝试往阻塞里添加节点
        Node pred = tail;
    	//如果pred也就是tail！=null，说明已经初始化过了
    	//PS:这下面的代码其实和enq的那段代码逻辑基本一样
        if (pred != null) {
            //设置当前node节点的前驱
            node.prev = pred;
            //CAS设置tail
            if (compareAndSetTail(pred, node)) {
                //设置pred的后继节点
                pred.next = node;
                //返回新加的节点
                return node;
            }
        }
    //到了这一步，只能说明阻塞队列没有初始化，或色是CAS失败，也就是有线程在竞争这个资源
    //enq方法可以回顾上面的分析
        enq(node);
        return node;
    }
```
### acquireQueued方法

```java
//返回是否打断
final boolean acquireQueued(final Node node, int arg) {
 	//标志是否获得成功，默认失败
    boolean failed = true;
    try {
        //标志是否被打断，默认没有
        boolean interrupted = false;
        //死循环
        for (;;) {
            //获得node节点的前驱
            final Node p = node.predecessor();
            //如果node的前驱p是head，说明可以尝试去获得锁
            if (p == head && tryAcquire(arg)) {
              //进入这里，获得成功，设置head
                  setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            //获得锁失败要么是p是不是头节点，没有资格获得锁，要么就是p是头节点，但是被别的线程抢先获得锁了（可能是非公平锁被抢占了），就判断是否要挂起线程
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```



### shouldParkAfterFailedAcquire方法

```java
/*
传进来2个参数，一个node的前驱结点pred，一个node节点
*/
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
   	//获得pred的waitStatus的状态
    int ws = pred.waitStatus;
    //如果是Node.SIGNAL,也就是-1,那么就说明当前节点可以被阻塞
    if (ws == Node.SIGNAL)
        return true;
    // >0 就是取消状态,说明当前节点前面的节点已经取消了,就要往前找到一个没有被取消的节点
    if (ws > 0) {
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    //设置前一个节点为SIGNAL状态,下一次进来的时候就会return
    } else {
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

###   parkAndCheckInterrupt方法

```java
private final boolean parkAndCheckInterrupt() {
    LockSupport.park(this);//阻塞线程
    //线程被重新唤醒后执行
    return Thread.interrupted();
}
```

### 关于加锁的流程图

1. 线程A进入,发现可以获得锁,那就获得锁,并没有对阻塞队列做任何处理
2. 线程B进入,不能获得锁,那么就要加入到阻塞队列中

   >  发现阻塞队列还没有初始化,那就先初始化,如下图

<img src="https://www.javadoop.com/blogimages/AbstractQueuedSynchronizer/aqs-1.png" alt="aqs-1" style="zoom:50%;" />

>  阻塞队列初始化完成,就将线程B加入到阻塞队列中,同时设置前面节点的waitStatus为-1,表示后继结点需要	被唤醒。

<img src="https://www.javadoop.com/blogimages/AbstractQueuedSynchronizer/aqs-2.png" alt="aqs-2" style="zoom:50%;" />

3. 线程C加入，不能获得锁，那就加入到阻塞队列，发现阻塞队列已经初始化，那就直接加到队尾，比设置前驱节点的waitStatus为-1，表示线程C需要被唤醒

<img src="https://www.javadoop.com/blogimages/AbstractQueuedSynchronizer/aqs-3.png" alt="aqs-3" style="zoom:50%;" />





## unlock解锁过程

```java
public void unlock() {
    sync.release(1);
}
```

### release方法

```java
public final boolean release(int arg) {
    //锁被完全释放了,唤醒阻塞队列中的节点
    if (tryRelease(arg)) {
        Node h = head;
        /*
        h == null Head还没初始化。初始情况下，head == null，第一个节点入队，Head会被初始化一个虚拟节		点。所以说，这里如果还没来得及入队，就会出现head == null 的情况。
		h != null && waitStatus == 0 表明后继节点对应的线程仍在运行中，不需要唤醒。
		h != null && waitStatus < 0 表明后继节点可能被阻塞了，需要唤醒。
        */
        if (h != null && h.waitStatus != 0)
          	//解除阻塞
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

### tryRelease方法

```java
//返回true,完全释放锁,返回false,没有完全释放锁
protected final boolean tryRelease(int releases) {
  	//主要是针对可重入锁的次数
    int c = getState() - releases;
	//当前线程不是持有锁的线程，抛出异常
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
   //因为是可重入锁,判断是否完全释放锁
    boolean free = false;
    //c==0 锁完全释放了,打标记,设置拥有线程为null
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    //设置state
    setState(c);
    return free;
}
```

### unparkSuccessor方法

```java
private void unparkSuccessor(Node node) {

    int ws = node.waitStatus;
    //即-1 ,那就设置为0
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    Node s = node.next;
    //如果下个节点是null或者下个节点被cancelled，就找到队列最开始的非cancelled的节点
    if (s == null || s.waitStatus > 0) {
        s = null;
        /*
        倒着遍历是因为,在addWaiter方法中,由于是双向链表,那么必然会有1->2,2->1的过程
       		 node.prev = pred;
             pred.next = node;
             上面两句是来自于addWaiter方法,我们可以看到是先设置前驱指针,然后设置后继指针,
             也就是在并发情况下,可能存在只设置了前驱指针,当腰设置后继的时候,线程就被切换了,导致还没有设			置.
             那么,这里的倒着遍历就可以理解了,防止没有遍历到所有的元素.
        */
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    //解除一个阻塞线程
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

## Condition相关

操作系统中有一个关于**生产者-消费者**的例子,在java中可以利用condition实现

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class BoundedBuffer {
    final Lock lock = new ReentrantLock();
    // condition 依赖于 lock 来产生
    final Condition notFull = lock.newCondition();
    final Condition notEmpty = lock.newCondition();

    final Object[] items = new Object[100];
    int putptr, takeptr, count;

    // 生产
    public void put(Object x) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length)
                notFull.await();  // 队列已满，等待，直到 not full 才能继续生产
            items[putptr] = x;
            if (++putptr == items.length) putptr = 0;
            ++count;
            notEmpty.signal(); // 生产成功，队列已经 not empty 了，发个通知出去
        } finally {
            lock.unlock();
        }
    }

    // 消费
    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0)
                notEmpty.await(); // 队列为空，等待，直到队列 not empty，才能继续消费
            Object x = items[takeptr];
            if (++takeptr == items.length) takeptr = 0;
            --count;
            notFull	.signal(); // 被我消费掉一个，队列 not full 了，发个通知出去
            return x;
        } finally {
            lock.unlock();
        }
    }
}
```

这其中的newCondition方法如下

```java
final ConditionObject newCondition() {
    return new ConditionObject();
}
```

其中ConditionObject是AQS的一个内部类，有两个比较重要的常量

```java
private transient Node firstWaiter;
private transient Node lastWaiter;
```

我们知道在AQS中存在着一个阻塞队列（本质上是双向链表），里面存放想要获得锁但是还没有获得锁的线程。

在我们使用condition的时候，引入一个新的队列，叫条件队列，这是一个单向链表(可以查看Node类，里面有一个nextWaiter属性从侧面印证了这一点)。

### 关于nextWaiter属性

其实**nextWaiter**的作用并不只是表示条件队列，在AQS的独占锁和共享锁的模式下，用于表示这两种不同的节点。

```java
// 共享模式
static final Node SHARED = new Node();
// 独占模式
static final Node EXCLUSIVE = null;
// 其他模式
// 其他非空值：条件等待节点（调用Condition的await方法的时候）
```





**下图的condition1和condition2都是条件队列。**

![condition-2](https://www.javadoop.com/blogimages/AbstractQueuedSynchronizer-2/aqs2-2.png)

### 大致流程

1. 调用await，将一个线程放入条件队列。
2. 某一个线程调用signal，唤醒一个线程，从条件队列中取出队首的放入阻塞队列（其实在java中称为Sync队列）。

### await方法

```java
public final void await() throws InterruptedException {
    //清除中断信息，如果当前已经中断，则抛出异常
    if (Thread.interrupted())
        throw new InterruptedException();
    //将当前节点加入到等待队列
    Node node = addConditionWaiter();
    //这个方法的作用就是完全释放锁，返回state次数
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    //比较重要的一个while，下面方法的分析
    while (!isOnSyncQueue(node)) {
        /*
        挂起线程
       	在第一次进入的时候，isOnSyncQueue是false，表示node还在条件队列中，那么需要唤醒，所以从
       	流程上来说，只有唤醒了才能进行下面的代码，所以下面分析signal方法
        */
        LockSupport.park(this);
        /*
        在经过signal之后，同步队列中被放入一个线程，这个线程总会有被重新激活的时候，如果重新激活了，			就会执行下面代码
        */
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    //之前释放了锁，现在要重新获得
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    /*checkInterruptWhileWaiting中，我们判断如果中断发生在signal前也会将节点加入到阻塞队列，但这个是		没有设置nextWaiter=null的，在doSignal中是设置了的
    */
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

### 带超时机制的await

```java
/*
加入了time，引入了超时机制，意思是如果超时了，就会主动加入到阻塞队列
*/
public final boolean await(long time, TimeUnit unit)
        throws InterruptedException {
    long nanosTimeout = unit.toNanos(time);
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
    final long deadline = System.nanoTime() + nanosTimeout;
    boolean timedout = false;
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        if (nanosTimeout <= 0L) {
            timedout = transferAfterCancelledWait(node);
            break;
        }
        if (nanosTimeout >= spinForTimeoutThreshold)
            LockSupport.parkNanos(this, nanosTimeout);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
        nanosTimeout = deadline - System.nanoTime();
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null)
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
    return !timedout;
}
```

### addConditionWaiter方法

```java
private Node addConditionWaiter() {
    Node t = lastWaiter;
    /*
    如果t==null 说明条件队列还没有元素
     t.waitStatus != Node.CONDITION，可能存在cancel的节点，那就要清理cancel节点
    */
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
      	//lastWaiter可能有变化，重新获得
        t = lastWaiter;
    }
    //封装当前线程节点
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
   	//t==null 说明条件队列还没有元素，那就设置第一个节点元素
    if (t == null)
        firstWaiter = node;
    //否则就在尾部接一个上去
    else
        t.nextWaiter = node;
    //更新尾节点
    lastWaiter = node;
    return node;
}
```

### unlinkCancelledWaiters方法

```java
//这个就是清理无效节点的操作，就是一个普通链表的删除操作
private void unlinkCancelledWaiters() {
    Node t = firstWaiter;
    Node trail = null;
    while (t != null) {
        Node next = t.nextWaiter;
        if (t.waitStatus != Node.CONDITION) {
            t.nextWaiter = null;
            if (trail == null)
                firstWaiter = next;
            else
                trail.nextWaiter = next;
            if (next == null)
                lastWaiter = trail;
        }
        else
            trail = t;
        t = next;
    }
}
```

### fullyRelease方法

```java
final int fullyRelease(Node node) {
    //标记是否释放失败，默认为失败
    boolean failed = true;
    try {
        //获得state的值
        int savedState = getState();
        //一次性释放，不再每次减1，这次是直接尝试减state
        if (release(savedState)) {
            failed = false;
            return savedState;
        } 
        //如果当前是没有获得锁的，就抛出异常
        else {
            throw new IllegalMonitorStateException();
        }
    } finally {
        //设置节点状态时CANCELLED
        if (failed)
            node.waitStatus = Node.CANCELLED;
    }
}
```

### isOnSyncQueuef方法

```java
/*
这个方法是判断node是否在同步队列（也就是我一直说的阻塞队列）
*/
final boolean isOnSyncQueue(Node node) {
    /*node.waitStatus == Node.CONDITION ，很明显还是条件队列中节点
    node.prev == null，pre属性是阻塞队列的一个属性，我们在前面说过，将一个节点加入到阻塞队列这个双		向链表，是个非原子操作，是先设置prev，再设置next，所以只需要判断prev是否为null，也能判断是否进入阻	 塞队列
   */
   if (node.waitStatus == Node.CONDITION || node.prev == null)
        return false;
    //同上面所说，如果next都有值了，那么就进入阻塞队列了
    if (node.next != null) 
        return true;
    return findNodeFromTail(node);
}
```

### findNodeFromTail方法

```java
//这个方法就是从尾节点开始遍历阻塞队列，判断node是否在阻塞队列中
private boolean findNodeFromTail(Node node) {
    Node t = tail;
    for (;;) {
        if (t == node)
            return true;
        if (t == null)
            return false;
        t = t.prev;
    }
}
```

### signal方法

```java
public final void signal() {
    //要调用signal就肯定要持有锁才行，就在这里判断，如果没有锁，就抛出异常
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    //如果firstWaiter==null,也就是条件队列没有内容,自然不用唤醒
    if (first != null)
        doSignal(first);
}
```

### isHeldExclusively方法

```java
//方法很简单，就是判断当前线程是不是持有锁的线程
protected final boolean isHeldExclusively() {
    return getExclusiveOwnerThread() == Thread.currentThread();
}
```

### doSignal方法

```java
private void doSignal(Node first) {
    do {
        //如果当前条件队列只有一个元素,那就清空
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
		//断开节点
        first.nextWaiter = null;
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
}
```

### 顺带说一下doSignalAll方法

```java
//可以看到与doSignal方法最大的区别就是doSignalAll是将所有能够加入到阻塞队列中的节点都加进去
private void doSignalAll(Node first) {
    lastWaiter = firstWaiter = null;
    do {
        Node next = first.nextWaiter;
        first.nextWaiter = null;
        transferForSignal(first);
        first = next;
    } while (first != null);
}
```

### transferForSignal方法

```java
final boolean transferForSignal(Node node) {
    //如果将当前节点从CONDITION设置为0失败,那么就会返回while语句,继续寻找一个新的节点唤醒,也就是说如果失败,那么就放弃这个节点
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;
    //进入到这里说明说明已经设置条件队列中的第一个元素的status是0,那么就把这个节点后加入到阻塞队列
    //注意,返回的p是尾节点的前驱节点,并不是尾节点
    Node p = enq(node);
    
    int ws = p.waitStatus;
	//大于0就说明是CACAELLED,或者就需要把waitStatus设置为SIGNAL,以方便在阻塞队列中的唤醒
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
      	//唤醒当前线程
        LockSupport.unpark(node.thread);
    return true;
}
```

关于这段代码的 `  LockSupport.unpark(node.thread);`多说一句。

这边的两个判断条件一定概率上处于优化作用，因为就算不执行，那么在signal之后，也会将一个等待队列中的节点加入到阻塞队列。在阻塞队列中的节点同样有希望解除挂起状态，重新获得锁，继续执行await中剩下的代码。这里就是直接唤醒，减少了再去阻塞队列中激活线程的一步。

### checkInterruptWhileWaiting方法

```java
//判断是否被中断
/*
三种不同的返回值
REINTERRUPT： 代表 await 返回的时候，需要重新设置中断状态
THROW_IE：	 代表 await 返回的时候，需要抛出 InterruptedException 异常
0 ：			 说明在 await 期间，没有发生中断
*/
private int checkInterruptWhileWaiting(Node node) {
    return Thread.interrupted() ?
        (transferAfterCancelledWait(node) ? THROW_IE : REINTERRUPT) :
        0;
}
```

### transferAfterCancelledWait方法

```java
final boolean transferAfterCancelledWait(Node node) {
  	//如果设置成功，说明中断是在signal前发生的，因为signal会将waitStatus设为0
    if (compareAndSetWaitStatus(node, Node.CONDITION, 0)) {
       //也会将节点加入到阻塞队列
        enq(node);
        return true;
    }
    //要等到node加入到阻塞队列，才产生返回
    while (!isOnSyncQueue(node))
        Thread.yield();
    return false;
}
```

> transferAfterCancelledWait方法并不在ConditionObject中定义，而是由AQS提供。这个方法根据是否中断发生时，是否有signal操作来“掺和”来返回结果。方法调用CAS操作将node的waitStatus从CONDITION设置为0，如果成功，说明当中断发生时，说明没有signal发生（signal的第一步是将node的waitStatus设置为0），在调用enq将线程放入Sync队列后直接返回true，表示中断先于signal发生，即中断在await等待过程中发生，根据await的语义，在遇到中断时需要抛出中断异常，返回true告诉上层方法返回THROW_IT，后续会根据这个返回值做抛出中断异常的处理。

> 如果CAS操作失败，是否说明中断后于signal发生呢？只能说这时候我们不能确定中断和signal到底谁先发生，只是在我们做CAS操作的时候，他们俩已经都发生了（中断->interrupted检测->signal->CAS，或者signal->中断->interrupted检测->CAS都有可能），这时候我们无法判断到底顺序是怎样，这里的处理是不管怎样都返回false告诉上层方法返回REINTERRUPT，当做是signal先发生（线程被signal唤醒）来处理，后续根据这个返回值做“补上”中断的处理。在返回false之前，我们要先做一下等待，直到当前线程被成功放入Sync锁等待队列。
>
> 源自：https://www.cnblogs.com/go2sea/p/5630355.html

### reportInterruptAfterWait方法

```java
//按照之前定义的返回，选择抛异常还是处理中断
private void reportInterruptAfterWait(int interruptMode)
    throws InterruptedException {
    if (interruptMode == THROW_IE)
        throw new InterruptedException();
    else if (interruptMode == REINTERRUPT)
        selfInterrupt();
}
```

## 共享锁

上面的关于ReentrantLock的内容和condition都是关于独占锁的。AQS中还存在着共享锁，下面会简单分析一下共享锁。

先说明一下共享锁的特点（个人看法）

1. 一开始我有点无法理解共享锁，但是可以类比概念上的读写锁，这样就好理解了。我们知道读锁是可以多次访问的，也就是说是个共享锁。**注意：state的值不会改变。**那么AQS判断是否可以拥有共享锁是通过tryAcquireShared方法来判断的，如果>=0,那么久允许获取。在CountDownLatch中。这个方法一个简单的实现就是直接判断state是不是为0。我们可以想象只有读锁的环境下，state的值永远是1，那么也就对应了任何读操作都能获得锁。

下面简单分析一下共享锁，这里面的代码写的非常巧妙，有很多不同判断，有点难以理解。

共享锁的内容其实和独占锁非常类似，可以从方法名上就可以看出。

| 独占锁                                      | 共享锁                                            |
| ------------------------------------------- | ------------------------------------------------- |
| tryAcquire(int arg)                         | tryAcquireShared(int arg)                         |
| tryAcquireNanos(int arg, long nanosTimeout) | tryAcquireSharedNanos(int arg, long nanosTimeout) |
| acquire(int arg)                            | acquireShared(int arg)                            |
| acquireQueued(final Node node, int arg)     | doAcquireShared(int arg)                          |
| acquireInterruptibly(int arg)               | acquireSharedInterruptibly(int arg)               |
| doAcquireInterruptibly(int arg)             | doAcquireSharedInterruptibly(int arg)             |
| doAcquireNanos(int arg, long nanosTimeout)  | doAcquireSharedNanos(int arg, long nanosTimeout)  |
| release(int arg)                            | releaseShared(int arg)                            |
| tryRelease(int arg)                         | tryReleaseShared(int arg)                         |
| -------------                               | doReleaseShared()                                 |

### acquireShared方法

```java
//获得锁
public final void acquireShared(int arg) {
   /* tryAcquireShared在AQS中并没有具体的实现，只是抛出了一个异常，在CountDownLatch，Semaphore，	ReentrantReadWriteLock中有实现，部分类后面后介绍
    说明一下tryAcquireShared的返回值
    如果该值小于0，则代表当前线程获取共享锁失败
    如果该值大于0，则代表当前线程获取共享锁成功，并且接下来其他线程尝试获取共享锁的行为很可能成功
    如果该值等于0，则代表当前线程获取共享锁成功，但是接下来其他线程尝试获取共享锁的行为会失败
    */
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}
```

### doAcquireShared方法

```java
/*
这个方法的作用就是获取共享锁
*/
private void doAcquireShared(int arg) {
    //与独享锁的第一个不同，Node.SHARED和Node.EXCLUSIVE的区别，本质上是区分两种模式
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            //第二个不同
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    //这个方法很重要
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

### setHeadAndPropagate方法

```java
/*
共享锁的特点就是能有多个线程拥有锁，所以如果你当前线程能够拥有锁，那么就需要唤醒阻塞队列中同同样能获得锁的系节点
*/
private void setHeadAndPropagate(Node node, int propagate) {
    Node h = head; // Record old head for check below
    //只有这个地方会修改head，与doReleaseShared中的跳出循环条件有关联
    setHead(node);
/*
1.propagate > 0 表示调用方指明了后继节点需要被唤醒
2.头节点后面的节点需要被唤醒（waitStatus<0），不论是老的头结点还是新的头结点
*/
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        Node s = node.next;
        //如果当前节点的后继节点是共享类型或者没有后继节点，则进行唤醒
        //这里可以理解为除非明确指明不需要唤醒（后继等待节点是独占类型），否则都要唤醒
        if (s == null || s.isShared())
            doReleaseShared();
    }
}
```

### doReleaseShared方法

```java
/*
真正的唤醒线程的地方
这个方法有两个地方会运行
1. acquireShared -> doAcquireShared -> setHeadAndPropagate 也就是获得锁的时候，会将阻塞队列中同样有机会的线程唤醒
2.releaseShared 释放锁的时候，同样会唤醒其他所有能够获得锁的线程
*/
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        //唤醒前提，阻塞队列中还有元素
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            /*
            	关于这个，太细了
            	我们可以看到这个语句由两部分
            	1. 判断ws是否等于0，等于0有2种情况，head=tail下ws是0，但这不可能
        			那就只可能是head后面刚添加节点，本该调用shouldParkAfterFailedAcquire修改head					的ws为-1，但是还没有执行的时候，会出现短暂的ws==0
        		2. 设置ws为Node.PROPAGATE，只有失败了才会continue，那么什么时候会失败？
        			唯一的解释就是后面的节点刚好执行了本该调用shouldParkAfterFailedAcquire修改						head的ws为-1
        		这种真的是只有极端的并发情况下才会产生的结果
            */
            else if (ws == 0 &&
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        /*
        这个是判断当前的head节点有没有改变，只有setHeadAndPropagate中才会改变，如果没有改变就会返回
        这边可以理解为一个优化，因为我们在上面已经唤醒了一个线程，那么本该可以返回，因为已经唤醒了一个线		程，那么唤醒第二个线程可以不用再操心了，由新唤醒的节点接替唤醒下一个节点的任务，但是这里判断了			head，那么如果新唤醒的线程已经变成了新的head节点，那么就会导致再次循环，也就是会有多个线程一起执		行唤醒线程操作，加快了唤醒速度。
        */if (h == head)                   // loop if head changed
            break;
    }
}
```

### releaseShared方法

```java
//这个方法就是释放锁的方法，具体的内部的两个方面上面都讨论过了
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

### 共享锁和独占锁的区别

- 当AQS的子类实现独占功能时，如ReentrantLock，资源是否可以被访问被定义为：只要AQS的state变量不为0，并且持有锁的线程不是当前线程，那么代表资源不可访问。
- 当AQS的子类实现共享功能时，如CountDownLatch，资源是否可以被访问被定义为：只要AQS的state变量不为0，那么代表资源不可以为访问。

## CountDownLatch

CountDownLatch是一个同步工具类，用来协调多个线程之间的同步，或者说起到线程之间的通信。

CountDownLatch 这个类是比较典型的 AQS 的共享模式的使用，下面会对这个类具体的说明，如果看懂了上面的共享锁，CountDownLatch应该还是比较好理解的。

具体例子

```java

class Driver2 { // ...
    void main() throws InterruptedException {
        //创建
        CountDownLatch doneSignal = new CountDownLatch(N);
        //使用线程池
        Executor e = Executors.newFixedThreadPool(8);
        // 创建 N 个任务，提交给线程池来执行
        for (int i = 0; i < N; ++i) // create and start threads
            e.execute(new WorkerRunnable(doneSignal, i));

        // 等待所有的任务完成，这个方法才会返回
        doneSignal.await();           // wait for all to finish
    }
}

class WorkerRunnable implements Runnable {
    private final CountDownLatch doneSignal;
    private final int i;

    WorkerRunnable(CountDownLatch doneSignal, int i) {
        this.doneSignal = doneSignal;
        this.i = i;
    }

    public void run() {
        try {
            doWork(i);
            // 这个线程的任务完成了，调用 countDown 方法
            doneSignal.countDown();
        } catch (InterruptedException ex) {
        } // return;
    }

    void doWork() { ...}
}
```

**这个类的核心方法就两个countDown和await**

```java
public void countDown() {
    sync.releaseShared(1);
}
/*****************************************/
  public final boolean releaseShared(int arg) {
        if (tryReleaseShared(arg)) {
        	//上面分析过了
            doReleaseShared();
            return true;
        }
        return false;
    }
/*
重写了tryReleaseShared可以看到每次countDown就state--
直到state减为0，就会唤醒所有await的线程
*/
protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
```

```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
/*******************这些代码前面都已经分析过了**************************/
  public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            //doAcquireSharedInterruptibly就是doAcquireShared方法，只不过处理了中断
            doAcquireSharedInterruptibly(arg);
    }

```

**最后再看一下构造函数**

```java
//传入了一个count，也就是需要执行count次countDown之后，才会唤醒
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```

## CyclicBarrier

CyclicBarrier是一个同步辅助类，允许一组线程互相等待，直到到达某个公共屏障点 (下面用栅栏代替)。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。

CyclicBarrier和CountDownLatch的实现不同，主要是通过ReentrantLock和Condition条件队列实现的。对于这两个概念不懂的，可以看上面的介绍。

**一个例子**

```java
package thread;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
/**
* 模拟运动员
**/
public class MyThread extends Thread {
    private CyclicBarrier cyclicBarrier;
    private String name;

    public MyThread(CyclicBarrier cyclicBarrier, String name) {
        super();
        this.cyclicBarrier = cyclicBarrier;
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println(name + "开始准备");
        try {
            Thread.currentThread().sleep(5000);
            System.out.println(name + "准备完毕！等待发令枪");
            try {
                cyclicBarrier.await();
            } catch (BrokenBarrierException e) {            
                e.printStackTrace();
            }
        } catch (InterruptedException e) {

            e.printStackTrace();
        }
    }
}
//测试类
public class Test {
    public static void main(String[] args) {
        CyclicBarrier barrier = new CyclicBarrier(5, new Runnable() {

            @Override
            public void run() {
                System.out.println("发令枪响了，跑！");

            }
        });
        for (int i = 0; i < 5; i++) {
            new MyThread(barrier, "运动员" + i + "号").start();

        }

    }

}
```

介绍一下属性

```java
//设置是否打破栅栏
private static class Generation {
    boolean broken = false;
}
//独占锁
private final ReentrantLock lock = new ReentrantLock();
//条件队列
private final Condition trip = lock.newCondition();
//设置同有parties个线程要同步
private final int parties;
//到达栅栏后要执行的操作
private final Runnable barrierCommand;
//利用这个实现重复利用
private Generation generation = new Generation();
//还有count个未达到栅栏
private int count;
```

主要方法是await，这里面有两个，一个是带超时参数的，一个是不带超时参数的

```java
//区别就是传入dowait的参数不同
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
/***********************************************************************/
  public int await(long timeout, TimeUnit unit)
        throws InterruptedException,
               BrokenBarrierException,
               TimeoutException {
        return dowait(true, unit.toNanos(timeout));
    }
```

### breakBarrier方法

```java
private void breakBarrier() {
    generation.broken = true;
    count = parties;
    trip.signalAll();
}
```

### nextGeneration方法

```java
/*
这里主要做了3件事
1. 唤醒所有的线程
2. 重置count为所有线程的值，方便开启下一个generation
3. 再次初始化generation，这么做的原因是区分不同的generation
*/private void nextGeneration() {
    trip.signalAll();
    count = parties;
    generation = new Generation();
}
```

#### 核心方法dowait

```java
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,
           TimeoutException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        //获得当前generation
        final Generation g = generation;
		//如果栅栏被打破，抛出异常
        if (g.broken)
            throw new BrokenBarrierException();
		//如果有中断，打破栅栏，抛出异常
        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }
		//说明有一个线程到达栅栏，那就count减1
        int index = --count;
        //index == 0说明所有的线程都已经到达栅栏了，那么就可以唤醒所有的线程，执行任务了 
        if (index == 0) {  
            //判断是否执行任务成功
            boolean ranAction = false;
            try {
                //获得任务
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                ranAction = true;
                //开启新的generation
                nextGeneration();
                return 0;
            } finally {
              //如果ranAction = false，说明在执行command发生了问题，那就要打破栅栏，下面会抛出异常
                if (!ranAction)
                    breakBarrier();
            }
        }
		//如果index！=0 ，说明还有线程未到达栅栏，就会进入这个死循环
        for (;;) {
            try {
                //如果没有超时标志，那就普通的await
                if (!timed)
                    trip.await();
                //如果带超时标志，那就调用超时机制的await
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                //进入到这里说明线程已经被重新唤醒，发现是被打断唤醒的，那就打破栅栏，重新抛出异常
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    //不然就设置打段标志
                    Thread.currentThread().interrupt();
                }
            }
            //栅栏被打破，抛出异常
            if (g.broken)
                throw new BrokenBarrierException();
			//这个方法在我看来就是提供一个正常返回的途径，因为在使用中，好像并不会接收这个返回值
            //因为已经已经开启了一个新的generation，那就可以返回了
            if (g != generation)
                return index;
			//超时，同样抛出异常
            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {a
        lock.unlock();
    }
}
```

然后对于这个方法，打破栅栏一共有3种可能

1. 中断，某个等待的线程发生了中断，那么会打破栅栏，同时抛出 InterruptedException 异常；
2. 超时，打破栅栏，同时抛出 TimeoutException 异常；
3. 指定执行的操作抛出了异常，这个我们前面也说过。

## CountDownLatch和CyclicBarrier的区别

CountDownLatch: 一个或者多个线程，等待其他多个线程完成某件事情之后才能执行。

CyclicBarrier : 多个线程互相等待，直到到达同一个同步点，再继续一起执行。

**应用场景**

比如说跑步比赛，有5个运动员

在出发前，只有5个运动员都准备好了，才开始比赛，这就是CyclicBarrier，5个线程互相等待。

当比赛开始后，只有5个运动员都到达终点了，才能进行颁奖，这就是CountDownLatch，颁奖线程需要等待5个运动员线程。

## Semaphore

Semaphore也叫信号量，在JDK1.5被引入，可以用来控制同时访问特定资源的线程数量，通过协调各个线程，以保证合理的使用资源。
Semaphore内部维护了一组虚拟的许可，许可的数量可以通过构造函数的参数指定。

在我的理解中，Semaphore有点像线程池，它规定最大的资源数量。每次acquire就会减少资源数量，release会增加资源数量。

Semaphore是基于AQS共享锁的一种实现，同时也有像ReentrantLock一样的公平锁和非公平锁。

```java
//公平策略
protected int tryAcquireShared(int acquires) {
    for (;;) {
        //区别就在这，公平策略先判断阻塞队列是否有元素，如果有，就会直接进入阻塞队列，准备挂起线程
        if (hasQueuedPredecessors())
            return -1;
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

```java
//非公平策略
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

其他感觉也没什么好分析的了，如果看懂了前面所有，那么Semaphore就很容易理解，可以自己去阅读。

## 参考内容

[一行一行源码分析清楚AbstractQue uedSynchronizer](https://www.javadoop.com/post/AbstractQueuedSynchronizer)

[一行一行源码分析清楚AbstractQue uedSynchronizer(二)](https://www.javadoop.com/post/AbstractQueuedSynchronizer-2)

[一行一行源码分析清楚AbstractQue uedSynchronizer(三)](https://www.javadoop.com/post/AbstractQueuedSynchronizer-3)

[从ReentrantLock的实现看AQS的原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)

[逐行分析AQS源码(3)——共享锁的获取与释放](https://segmentfault.com/a/1190000016447307)

[AbstractQueuedSynchronizer源码解读](https://www.cnblogs.com/micrari/p/6937995.html)

[Java并发之CyclicBarrier](https://juejin.im/entry/596a05fdf265da6c4f34f2f9#comment)