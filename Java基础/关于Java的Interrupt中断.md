## 关于Java的Interrupt中断

[toc]

#### 概念

首先需要知道的是每一个线程都有一个中断状态位，默认值是false。

然后当调用一个线程的`interrupt()`方法之后，会有两种情况： 

1. 第一种，线程处于正常的运行状态下，线程的中断状态位会被改变，会由false变为true，但是线程并**不会停止运行**。
2. 如果线程正处于阻塞的状态(sleep, wait, join 等状态)，那么线程将会退出阻塞，并且清空状态位（也就是设位false），并抛出 `InterruptException`

所以如果想要退出线程：

如果是正在运行的线程，可以通过监听状态位来变化选择退出循环，结束线程。

如果是阻塞的线程，可以通过捕捉异常，然后选择退出循环以结束线程。

#### 例子

可以通过下面的代码来看阻塞线程调用`interrupt()`方法的情况。

```java
package com.yiqzq;

public class InterruptTest {
    //这里用来打印消耗的时间
    private static long time = 0;

    private static void resetTime() {
        time = System.currentTimeMillis();
    }

    private static void printContent(String content) {
        System.out.println(content + "     时间：" + (System.currentTimeMillis() - time));
    }

    public static void main(String[] args) {

        test1();

    }

    private static void test1() {

        Thread1 thread1 = new Thread1();
        thread1.start();

        //延时3秒后interrupt中断
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        thread1.interrupt();
        printContent("执行中断");

    }

    private static class Thread1 extends Thread {

        @Override
        public void run() {

            resetTime();

            int num = 0;
            while (true) {
                if (isInterrupted()) {
                    printContent("当前线程 isInterrupted");
                    break;
                }

                num++;
                //使线程阻塞
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    System.out.println("产生中断");
                }
                if (num % 100 == 0) {
                    printContent("num : " + num);
                }
            }

        }

    }

}


```

可以看到结果如图

![](C:\Users\39268\Desktop\QQ截图20200219215454.png)



从这个图中，我们可以看到在main函数执行了`interrupt()`方法之后，在while循环之中，`isInterrupted()`的返回值是false，因此并没有调用里面的内容。这由此可以印证，对阻塞中的线程调用`interrupt()`，会对标志位重置，并且我们也可以看到`产生中断`这个内容是由catch块产生的。



为了对比正常运行线程的效果，可以将下面的`sleep`删除以观察效果。