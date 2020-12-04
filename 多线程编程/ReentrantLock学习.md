# ReentrantLock学习

[toc]

## 公平锁和非公平锁的区别

前面在[AQS学习](https://blog.csdn.net/yiqzq/article/details/105027963)一文中已经提到过了公平锁的整个流程,所以现在来看一下非公平锁

首先是关于ReentrantLock的默认表现形式。

**总共有2个构造器，由下面的代码可知，默认实现的是非公平锁，但是也可以传入参数控制是公平锁还是非公平锁。**

```java
public ReentrantLock() {
    sync = new NonfairSync();
}
/*********************************************************************/
public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

**下面来看一下非公平锁的具体实现**

主要区别就是下面三个方法

```java
final void lock() {
    //相较于公平锁，当调用lock的时候就先尝试CAS获得锁
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        //如果失败就和公平锁一样进入acquire方法，既然一样就不写了
        acquire(1);
}
```

```java
//进入acquire，首先会尝试tryAcquire
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}
```

```java
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        //相比于公平锁，不再判断阻塞队列，直接擦尝试获得锁
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
       //省略不重要的
    }
    return false;
}
```

所以，大致就两处不同，就是非公平锁第一次进入lock的时候会先尝试获得锁，然后再nonfairTryAcquire中，不再判断阻塞队列中的元素，也是直接尝试获得锁。别的地方就基本一样，获得锁失败之后进入阻塞队列。