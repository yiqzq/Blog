# Fork/Join的学习笔记

[toc]

## Fork/Join的出现

在JDK5中出现了concurrent 包，这里面有Executors框架，增强普通的线程。但是并没有很好的利用现代计算机多核的优势。在JDK7中，为了利用计算机多核的优势，于是出现了Fork/Join框架。这是一个并行的框架，有效利用了多核CPU的计算能力。

## Fork/Join的介绍

Fork/Join里面有一个很重要的算法，工作窃取算法。

工作窃取（work-stealing）算法是指某个线程从其他队列里窃取任务来执行。当⼀个线程窃取另⼀个线程的时候，为了减少两个任务线程之间的 竞争，我们通常使⽤双端队列来存储任务。被窃取的任务线程都从双端队列的头部 拿任务执⾏，⽽窃取其他任务的线程从双端队列的尾部执⾏任务。

![img](https://static001.infoq.cn/resource/image/5f/ca/5fcb634bbd48d722952ff2c9340892ca.png)

使用工作窃取算法能够最大化利用CPU资源加快任务完成的速度。	

Fork/Join采用的是分治的思想，把一个大任务拆分成子任务，最后通过合并子任务获得最终的运行结果。

![img](https://static001.infoq.cn/resource/image/2b/09/2be16e00a8ee6f6c7b738817f003e609.png)

## 使用Fork/Join

Fork/Join涉及到两个类，ForkJoinTask和ForkJoinPool 。

ForkJoinTask的核心方法就是fork和join。

```java
//其实fork()只做了⼀件事，那就是把任务推⼊当前⼯作线程的⼯作队列⾥。
public final ForkJoinTask<V> fork() {
        Thread t;
     	//先判断当前线程是不是ForkJoin的工作线程
        if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread)
            //如果是的话，对应的工作线程的workQueue里面添加当前ForkJoinTask任务
            ((ForkJoinWorkerThread)t).workQueue.push(this);
        else
            //否则就在ForkJoinPool的公共队列中加入当前任务
            ForkJoinPool.common.externalPush(this);
     	//返回自身
        return this;
    }
```

```java
//用于得到当前任务的运行结果
public final V join() {
    int s;
     // doJoin()⽅法来获取当前任务的执⾏状态
    if ((s = doJoin() & DONE_MASK) != NORMAL)
        reportException(s);
      // 任务正常完成，获取返回值 
    return getRawResult();
}
```

```java
private int doJoin() {
    int s; Thread t; ForkJoinWorkerThread wt; ForkJoinPool.WorkQueue w;
     // 先判断任务是否执⾏完毕，执⾏完毕直接返回结果（执⾏状态） 
    return (s = status) < 0 ? s :
     // 如果没有执⾏完毕，先判断是否是ForkJoinWorkThread线程 
        ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) ?
            // 如果是，先判断任务是否处于⼯作队列顶端（意味着下⼀个就执⾏它）    
            // tryUnpush()⽅法判断任务是否处于当前⼯作队列顶端，是返回true       
            // doExec()⽅法执⾏任务 
        (w = (wt = (ForkJoinWorkerThread)t).workQueue).
            // 如果是处于顶端并且任务执⾏完毕，返回结果 
        tryUnpush(this) && (s = doExec()) < 0 ? s :
     	 // 如果不在顶端或者在顶端却没未执⾏完毕，那就调⽤awitJoin()执⾏任务       
    	// awaitJoin()：使⽤⾃旋使任务执⾏完成，返回结果   
        wt.pool.awaitJoin(w, this, 0L) :
     // 如果不是ForkJoinWorkThread线程，执⾏externalAwaitDone()返回任务结果 
        externalAwaitDone();
}
```

通常情况下，在创建任务的时候我们⼀般不直接继承ForkJoinTask，⽽是继承它的 ⼦类RecursiveAction和RecursiveTask。
两个都是ForkJoinTask的⼦类，RecursiveAction可以看做是⽆返回值的 ForkJoinTask，RecursiveTask是有返回值的ForkJoinTask。

```java
@sun.misc.Contended
public class ForkJoinPool extends AbstractExecutorService { 
  	 // 任务队列，这里有保存JoinWorkerThread，下面会简单分析WorkQueue
    volatile WorkQueue[] workQueues;     // main registry
     // 线程的运⾏状态     
    volatile int runState;   
    //工厂类
    final ForkJoinWorkerThreadFactory factory;
    //公共池子，意味着可以不创建ForkJoinPool就可以进行分治并发编程
    static final ForkJoinPool common;
    //构造函数
    private ForkJoinPool(int parallelism,
                         ForkJoinWorkerThreadFactory factory,
                         UncaughtExceptionHandler handler,
                         int mode,
                         String workerNamePrefix) {
        this.workerNamePrefix = workerNamePrefix;
        this.factory = factory;
        this.ueh = handler;
        this.config = (parallelism & SMASK) | mode;
        long np = (long)(-parallelism); // offset ctl counts
        this.ctl = ((np << AC_SHIFT) & AC_MASK) | ((np << TC_SHIFT) & TC_MASK);
    }
}
```

```java
  @sun.misc.Contended
    static final class WorkQueue {
     	//这个就是保存任务队列，WorkQueue的双端队列是通过数组实现的
        ForkJoinTask<?>[] array;  
        final ForkJoinPool pool;  
        final ForkJoinWorkerThread owner;
        //可以看到，WorkQueue关联了池子和线程
        WorkQueue(ForkJoinPool pool, ForkJoinWorkerThread owner) {
            this.pool = pool;
            this.owner = owner;
            base = top = INITIAL_QUEUE_CAPACITY >>> 1;
        }
```



## 使用例子

```java
public class ForkJoinCalculator implements Calculator {
    private ForkJoinPool pool;

    private static class SumTask extends RecursiveTask<Long> {
        private long[] numbers;
        private int from;
        private int to;

        public SumTask(long[] numbers, int from, int to) {
            this.numbers = numbers;
            this.from = from;
            this.to = to;
        }

        @Override
        protected Long compute() {
            // 当需要计算的数字小于6时，直接计算结果
            if (to - from < 6) {
                long total = 0;
                for (int i = from; i <= to; i++) {
                    total += numbers[i];
                }
                return total;
            // 否则，把任务一分为二，递归计算
            } else {
                int middle = (from + to) / 2;
                SumTask taskLeft = new SumTask(numbers, from, middle);
                SumTask taskRight = new SumTask(numbers, middle+1, to);
                taskLeft.fork();
                taskRight.fork();
                return taskLeft.join() + taskRight.join();
            }
        }
    }

    public ForkJoinCalculator() {
        // 也可以使用公用的 ForkJoinPool：
        // pool = ForkJoinPool.commonPool()
        pool = new ForkJoinPool();
    }

    @Override
    public long sumUp(long[] numbers) {
        return pool.invoke(new SumTask(numbers, 0, numbers.length-1));
    }
}
```



## 参考资料

[聊聊并发（八）——Fork/Join 框架介绍](https://www.infoq.cn/article/fork-join-introduction)

《深入浅出多线程》

[jdk1.8-ForkJoin框架剖析](https://www.jianshu.com/p/f777abb7b251)

[JUC源码分析-线程池篇（四）：ForkJoinPool - 1](https://www.jianshu.com/p/32a15ef2f1bf)

