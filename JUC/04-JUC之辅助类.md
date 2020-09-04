## JUC之辅助类

### 1、同步计数器

```Java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        //创建计数器初始值6，为每个线程分配一个计数器
        CountDownLatch countDownLatch = new CountDownLatch(6);

        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "go");
                //计数器减一
                countDownLatch.countDown();
            }).start();
        }

        //让下面的代码等待计数器走完了才能走
        countDownLatch.await();
        System.out.println("最后执行");
    }
}
```

CountDownLatch是一个同步工具类，用来协调多线程之间的同步。当计数器数值减为0时，所有受其影响而等待的线程将会被激活。



### 2、循环栅栏

```Java
public class CyclicBarrierDemo {
    public static void main(String[] args) throws InterruptedException {
        //创建循环栅栏，并且定义循环的次数和最后执行的语句
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {
            System.out.println("最后执行");
        });
        for (int i = 0; i < 7; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "go");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }

            }).start();
        }
    }
}
```

其作用就是多线程的进行阻塞，等待某一个临界值条件满足后，同时执行！假设有一个场景：每个线程代表一个跑步运动员，当运动员都准备好后，才一起出发，只要有一个人没有准备好，大家都等待！

CountDownLatch和CyclicBrrier的区别

* CountDownLatch: 一个线程(或者多个)， 等待另外N个线程完成某个事情之后才能执行。

* CyclicBrrier: N个线程相互等待，任何一个线程完成之前，所有的线程都必须等待。



### 3、信号量

```Java
public class SemaphoreDemo {
    public static void main(String[] args) {
        //创建Semaphore对象，并且模拟有三个资源
        Semaphore semaphore = new Semaphore(3);

        //模拟6个线程去争抢三个资源
        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                try {
                    //占用资源，资源--
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "线程抢到资源");
                    //模拟每个线程的占用时间
                    Thread.sleep(3000);
                    System.out.println(Thread.currentThread().getName() + "线程释放资源");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    //释放资源，资源++
                    semaphore.release();
                }
            }).start();
        }
    }
}
```

信号量主要用于多个共享资源互斥使用和用于并发线程数的控制。

acquire(获取)：当一个线程调用acquire操作时，它要么成功获取信号量，此时信号量减1，要么一种等待下去，直到有线程释放信号量或超时

release（释放）：信号量加1，然后唤醒等待的线程

如果new Semaphore(1)：相当于synchronized，一次只能进去一个线程，并且可以设定每个线程的等待时间。