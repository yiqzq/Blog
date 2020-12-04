# 深入理解Java虚拟机读书笔记(五）

[toc]

**动态类型语言支持--invokedynamic指令(没看懂,改日再看)**

执行引擎在执行Java代码的方式:**解释执行**（通过解释器执行）和**编译执行**（通过即时编译器产生本地代码执行）



指令集架构：**基于栈的指令集**（java字节码这一套）与**基于寄存器的指令集**（传统的x86）

**基于栈的指令集**的优点：

1. 可移植性，因为不依赖于寄存器
2. 代码相对紧凑
3. 编译器实现更加简单

缺点：

1. 慢

**前端编译器：对从程序到抽象语法树或中间字节码的生成的优化**

**关于Java自动拆箱和自动装箱的一个陷阱**

```java
public static void main(String[] args) {
    Integer a = 1;
    Integer b = 2;
    Integer c = 3;
    Integer d = 3;
    Integer e = 321;
    Integer f = 321;
    Long g = 3L;
    System.out.println(c == d);
    System.out.println(e == f);
    System.out.println(c == (a + b));
    System.out.println(c.equals(a + b));
    System.out.println(g == (a + b));
    System.out.println(g.equals(a + b));
/*
答案
true
false
true
true
true
false
*/
}
```

需要值得注意的有3点：

1. 在Integer和Long中，数据范围在[-128,127]之中的数都是static的
2. == 这个运算符在不遇到算术运算的情况下不会自动拆箱
3. equals()方法不处理数据转型的关系，看下源码就懂了

```java
public boolean equals(Object obj) {
    if (obj instanceof Integer) {//会先判断类型关系
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```

后端编译器：提前编译和即时编译（也没看懂，以后再看）

Java内存模型和多线程

1. volatile

2. 原子性、可见性与有序性

   可见性：除了volatile之外，Java还有两个关键字能实现可见性，它们是synchronized和final

3. 先行发生原则

**先行发生**是Java内存模型中定义的两项操作之间的偏序关系，比如说操作A先行发生于操作B，其实就是说在发生操作B之前，操作A产生的影响能被操作B 观察到，“影响”包括修改了内存中共享变量的值、发送了消息、调用了方法等

**Java内存模型自带的先行发生关系**

程序次序规则（Program Order Rule）：在一个线程内，按照控制流顺序，书写在前面的操作先行发生于书写在后面的操作。注意，这里说的是控制流顺序而不是程序代码顺序，因为要考虑分支、循 环等结构。

管程锁定规则（Monitor Lock Rule）：一个unlock操作先行发生于后面对同一个锁的lock操作。这里必须强调的是“同一个锁”，而“后面”是指时间上的先后。 

volatile变量规则（Volatile Variable Rule）：对一个volatile变量的写操作先行发生于后面对这个变量的读操作，这里的“后面”同样是指时间上的先后。

线程启动规则（Thread Start Rule）：Thread对象的start()方法先行发生于此线程的每一个动作。 

线程终止规则（Thread Termination Rule）：线程中的所有操作都先行发生于对此线程的终止检测，我们可以通过Thread::join()方法是否结束、Thread::isAlive()的返回值等手段检测线程是否已经终止 执行。
线程中断规则（Thread Interruption Rule）：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过Thread::interrupted()方法检测到是否有中断发生。 

对象终结规则（Finalizer Rule）：一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始。 

传递性（Transitivity）：如果操作A先行发生于操作B，操作B先行发生于操作C，那就可以得出
操作A先行发生于操作C的结论