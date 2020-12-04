#  Java 如何去检测一个死锁

[toc]

1. 先要去构造一个死锁

```java
/**
 * Created by lirong5 on 2016/5/24.
 */
public class SyncDeadLock{
    private static Object locka = new Object();
    private static Object lockb = new Object();

    public static void main(String[] args){
        new SyncDeadLock().deadLock();
    }

    private void deadLock(){
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (locka){
                    try{
                        System.out.println(Thread.currentThread().getName()+" get locka ing!");
                        Thread.sleep(500);
                        System.out.println(Thread.currentThread().getName()+" after sleep 500ms!");
                    }catch(Exception e){
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName()+" need lockb!Just waiting!");
                    synchronized (lockb){
                        System.out.println(Thread.currentThread().getName()+" get lockb ing!");
                    }
                }
            }
        },"thread1");

        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (lockb){
                    try{
                        System.out.println(Thread.currentThread().getName()+" get lockb ing!");
                        Thread.sleep(500);
                        System.out.println(Thread.currentThread().getName()+" after sleep 500ms!");
                    }catch(Exception e){
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName()+" need locka! Just waiting!");
                    synchronized (locka){
                        System.out.println(Thread.currentThread().getName()+" get locka ing!");
                    }
                }
            }
        },"thread2");

        thread1.start();
        thread2.start();
    }
}
```

2. 大概有三种方法可以去加测死锁

- 使用jps + jstack

1. 使用指令`jps -l`

<img src="https://i.loli.net/2020/04/03/LQAYBCeqfjgJEyV.png" alt="image-20200403133541010" style="zoom:80%;" />

2. 查看具体的进程，`jstack -l 6168`

<img src="https://i.loli.net/2020/04/03/ZCnudwc2v94JSHq.png" alt="image-20200403134010399" style="zoom:80%;" />

可以看到最下面有一行语句` Found 1 deadlock`,意思就是发现一个死锁。

- 可以使用jconsole工具

  命令行直接输入打开即可，选择要查看的进程

<img src="https://i.loli.net/2020/04/03/6wMIDi241KSePBL.png" alt="image-20200403134431554" style="zoom:67%;" />

进入主页面的线程选项，有一个检测死锁按钮

<img src="https://i.loli.net/2020/04/03/QkfPeoGK4Rs2t9q.png" alt="image-20200403134710378" style="zoom:80%;" />

我们可以看到有两个线程死锁了。

-  使用VisualVM检测死锁，在IDEA中自动集成了这个可视化工具

<img src="https://i.loli.net/2020/04/03/7JP5LghUkznSsrj.png" alt="image-20200403134848401" style="zoom:80%;" />

可以看到，选择对应的进程，系统会自动提示你有死锁线程，非常的方便。当然可以点击右上方的线程dump查看具体信息，其实也就是jstack之前展示过的消息。