# JUC之线程池

## 一、线程池概述

之前的创建线程过程为`Thread thread = new Thread()`，每一个线程都new出来的，众所周知每一次new对象都是对cpu的资源极大开销，为了避免这一情况，出现了线程池。线程池顾名思义，一次创建多个线程放到一个池子里，谁用谁去池子里取，用完再还回池子，避免了重复new对象的操作。

优势：线程池的工作只是控制运行的线程数量，处理过程中将认为放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量，超出数量的线程排队等候，等待其他线程执行完毕，再从队列中取出任务执行。

特点为：线程复用、控制最大并发数、管理线程



## 二、使用线程池

### 线程池架构

![image-20200818165655342](http://picture.youyouluming.cn/image-20200818165655342.png)



### 创建固定容量线程池

代码

```Java
public class ThreadPoolDemo {
    public static void main(String[] args) {
        //创建线程池，指定线程池中的线程5个
        ExecutorService executorService = Executors.newFixedThreadPool(5);

        try {
            //模拟10个需要使用线程的操作
            for (int i = 0; i < 10; i++) {
                executorService.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "线程执行");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //把线程放回池子
            executorService.shutdown();
        }
    }
}
```

结果

![image-20200818171004029](http://picture.youyouluming.cn/image-20200818171004029.png)

### 单例线程池

代码

```Java
public class ThreadPoolDemo02 {
    public static void main(String[] args) {
        //创建单例线程池，池里只有一个线程
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        try {
            //模拟10个需要使用线程的操作
            for (int i = 0; i < 10; i++) {
                executorService.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "线程执行");
                });
                Thread.sleep(400);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //把线程放回池子
            executorService.shutdown();
        }
    }
}
```

结果

![image-20200818172452438](http://picture.youyouluming.cn/image-20200818172452438.png)

一池一线程，一个任务一个任务的执行



### N线程池

代码

```Java
public class ThreadPoolDemo03 {
    public static void main(String[] args) {
        //创建线程池，可以根据请求数变化而变化
        ExecutorService executorService = Executors.newCachedThreadPool();

        try {
            //模拟10个需要使用线程的操作
            for (int i = 0; i < 10; i++) {
                executorService.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "线程执行");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //把线程放回池子
            executorService.shutdown();
        }
    }
}
```

结果

![image-20200818172820486](http://picture.youyouluming.cn/image-20200818172820486.png)

可以随着需求而自动变化线程池中的线程数

比如每个需求之前都暂停一下，结果就不一样了

代码

```Java
public class ThreadPoolDemo03 {
    public static void main(String[] args) {
        //创建线程池，可以根据请求数变化而变化
        ExecutorService executorService = Executors.newCachedThreadPool();

        try {
            //模拟10个需要使用线程的操作
            for (int i = 0; i < 10; i++) {
                Thread.sleep(1000);
                executorService.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "线程执行");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //把线程放回池子
            executorService.shutdown();
        }
    }
}
```

结果

![image-20200818173042048](http://picture.youyouluming.cn/image-20200818173042048.png)

当上一个线程使用完放回池中，下一个线程就可以接着用不用创建新的线程。


## 三、线程池的参数

**七大参数**

![image-20200818194801864](http://picture.youyouluming.cn/image-20200818194801864.png)

1. corePoolSize：线程池中常驻核心线程数
2. maximumPoolSize：线程中能够容纳同时执行的最大线程数，此值必须大于等于1。
3. keepAliveTime：多余的空闲线程存活时间，当线程池中的线程数量超过corePoolSize时并且空闲时间达到keepAliveTime时，多余线程会被销毁，直到只剩下corePoolSize个线程为止。
4. unit：keepAliveTime的单位。
5. workQueue：任务队列，被提交但尚未被执行的任务。
6. threadFactory：表示生产线程池中工作线程的线程工厂，用于创建线程，一般使用默认的。
7. handler：拒绝策略，表示当队列满了，并且工作线程大于等于线程池的最大线程池（maximumPoolSize）时如何来拒绝请求执行的Runnable的策略。

## 四、线程池的原理

三个创建线程池方法的源码

> ```
> newFixedThreadPool
> ```

![image-20200818174408181](http://picture.youyouluming.cn/image-20200818174408181.png)

> ```
> newSingleThreadExecutor
> ```

![image-20200818174426370](http://picture.youyouluming.cn/image-20200818174426370.png)

> ```
> newCachedThreadPool
> ```

![image-20200818174442624](http://picture.youyouluming.cn/image-20200818174442624.png)



**工作原理图**

![image-20200818210021139](http://picture.youyouluming.cn/image-20200818210021139.png)

**工作流程**

1. 创建线程池，开始等待
2. 当调用execute()方法添加一个任务请求后，线程池会做出以下判断
   1. 如果当前正在运行的线程数小于corePoolSize，马上创建线程执行该任务
   2. 如果当前运行的线程数大于或等于corePoolSize，那么将这个任务放到阻塞队列
   3. 如果这个时候阻塞队列满了且正在运行的线程数量小于maximumPoolSize，那么会创建非核心线程运行该任务
   4. 如果队列也满了且正在运行的线程数量大于等于maximumPoolSize，那么线程池会启动饱和拒绝策略来执行
3. 当线程完成任务时，会从队列取出下一个任务来执行
4. 当一个线程无事可做超过一定时间（keepAliveTime）时，线程会判断
   1. 如果当前运行线程大于corePoolSize，那么当前这个线程就会被停止
   2. 所有线程完成任务后，线程池会收缩到corePoolSize的大小



**实际开发中的配置**

问：在工作中上面三种创建线程池的方法使用哪一种最多？

答：一个都不用，使用自定义的

问：Executors中JDK已经提供了为什么不用？

答：参考阿里巴巴Java开发手册

![image-20200818232905532](http://picture.youyouluming.cn/image-20200818232905532.png)







## 五、拒绝策略

* AbortPolicy（默认）：直接抛出RejectedExecutionException异常阻止系统正常运行。
* CallerRunsPolicy：“调用者运行”一种调节机制，该策略不会抛弃任务，也不好抛出异常，而是将某些任务回退到调用者，从而降低新任务的流量
* DiscardOldestPolicy：抛弃队列中等待最久的任务，然后把当前任务加入队列中尝试再次提交当前任务。
* DiscardPolicy：该策略默默丢弃无法处理的任务，不予任何处理也不抛出任何异常。如果允许任务丢失，这时最好的一种策略



**代码演示各个拒绝策略**

> AbortPolicy 	直接抛出RejectedExecutionException异常阻止系统正常运行。

```Java
public class ThreadPoolDemo04 {
    public static void main(String[] args) {
        //自定义方式创建线程池
        ExecutorService executorService = new ThreadPoolExecutor(
                2,
                5,
                2L,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.AbortPolicy());


        try {
            for (int i = 0; i < 9; i++) {
                executorService.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "线程执行");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            executorService.shutdown();
        }
    }
}
```

此时线程池可容纳的最大线程数量为maximumPoolSize+阻塞队列（workQueue）中定义的容量=8，但是创建了9个线程，看看结果如何。

![image-20200818235137347](http://picture.youyouluming.cn/image-20200818235137347.png)

报的RejectedExecutionException（拒绝执行异常），时四大拒绝策略的默认策略AbortPolicy报出的异常，直接中断程序。



> CallerRunsPolicy  “调用者运行”一种调节机制，该策略不会抛弃任务，也不好抛出异常，而是将某些任务回退到调用者，从而降低新任务的流量

```java
public class ThreadPoolDemo04 {
    public static void main(String[] args) {
        //自定义方式创建线程池
        ExecutorService executorService = new ThreadPoolExecutor(
                2,
                5,
                2L,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.CallerRunsPolicy());


        try {
            for (int i = 0; i < 10; i++) {
                executorService.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "线程执行");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            executorService.shutdown();
        }
    }
}
```

结果

![image-20200819000632805](http://picture.youyouluming.cn/image-20200819000632805.png)

会把超出最大容量的线程返回给调用者，此时的调用者便是main方法



> DiscardOldestPolicy：抛弃队列中等待最久的任务，然后把当前任务加入队列中尝试再次提交当前任务。
>
> DiscardPolicy：该策略默默丢弃无法处理的任务，不予任何处理也不抛出任何异常。如果允许任务丢失，这时最好的一种策略
>
> 两个策略效果相同

```java
public class ThreadPoolDemo04 {
    public static void main(String[] args) {
        //自定义方式创建线程池
        ExecutorService executorService = new ThreadPoolExecutor(
                2,
                5,
                2L,
                TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(3),
                Executors.defaultThreadFactory(),
                new ThreadPoolExecutor.DiscardOldestPolicy());


        try {
            for (int i = 0; i < 10; i++) {
                executorService.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "线程执行");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            executorService.shutdown();
        }
    }
}
```

结果

![image-20200819001155892](http://picture.youyouluming.cn/image-20200819001155892.png)





补充

问：怎么定义maximumPoolSize的大小？

答：根据本机cpu核心数+1配置，获取本机核心数`Runtime.getRuntime().availableProcessors();`