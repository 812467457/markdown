# JUC之BlockingQueue（阻塞队列）

## 一、BlockingQueue概述

两种情况：

* 当队列是空的，从队列中获取元素的操作将会被阻塞
* 当队列是满的，从队列中获取元素的操作将会被阻塞

阻塞队列的好处：

* 阻塞也叫挂起，之前的wait就是挂起线程，等条件满足，被挂起的线程又会被唤醒。
* 而阻塞队列可以帮我们把何时阻塞何时需要唤醒的事情都给做了，不用我们自己关心阻塞和唤醒。

## 二、BlockingQueue常用方法

| 方法类型 | 抛出异常  | 特殊值   | 阻塞   | 超时              |
| -------- | --------- | -------- | ------ | ----------------- |
| 插入     | add(e)    | offer(e) | put(e) | offer(e,timeunit) |
| 移除     | remove(e) | poll()   | take() | poll(time,unit)   |
| 检查     | element() | peek()   | 不可用 | 不可用            |

* 抛出异常：
  * 当队列满时，再往队列里add插入元素会抛IllegalStateException: Queue full异常
  * 当队列为空，再从队列里remove移除元素会抛NoSuchElementException异常

* 特殊值：
  * 插入方法，成功返回true，失败false
  * 移除方法，成功返回元素，没有元素就返回null

* 阻塞：
  * 当阻塞队列满时，生产者线程继续往队列put元素，队列会一直阻塞生产者线程直到pull数据或响应中断退出。
  * 当阻塞队列空时，消费者线程试图从队列里take元素，队列会一直阻塞消费者线程直到队列可用
* 超时退出：当阻塞队列满时，队列会阻塞生产者线程一段时间，超过时间后生产者线程会自动退出



#### 抛出异常代码演示：

> add

```java 
public class BlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        //创建一个容量为3的阻塞队列
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
        //返回类型是boolean
        System.out.println(blockingQueue.add("a"));
        System.out.println(blockingQueue.add("c"));
        System.out.println(blockingQueue.add("d"));
        System.out.println(blockingQueue.add("d"));
    }
}
```

结果：

```shell
true
true
true
Exception in thread "main" java.lang.IllegalStateException: Queue full
	at java.util.AbstractQueue.add(AbstractQueue.java:98)
	at java.util.concurrent.ArrayBlockingQueue.add(ArrayBlockingQueue.java:312)
	at juc.BlockingQueueDemo.main(BlockingQueueDemo.java:16)
```



> remove

```Java
public class BlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        //创建一个容量为3的阻塞队列
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(blockingQueue.add("a"));
        System.out.println(blockingQueue.add("c"));
        System.out.println(blockingQueue.add("d"));

        //返回类型是E
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
    }
}
```

结果：

```shell
true
true
true
a
c
d
Exception in thread "main" java.util.NoSuchElementException
	at java.util.AbstractQueue.remove(AbstractQueue.java:117)
	at cn.yylm._04blockingQueue.BlockingQueueDemo.main(BlockingQueueDemo.java:18)
```



> element	检查队首元素是什么

```Java
public class BlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        //创建一个容量为3的阻塞队列
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(blockingQueue.add("a"));
        System.out.println(blockingQueue.add("c"));
        System.out.println(blockingQueue.add("d"));

        System.out.println(blockingQueue.element());
    }
}
```

结果

```shell
true
true
true
a
```

#### 特殊值代码演示

> offer 插入，此时插入失败不会报错，而是返回false

```java 
public class BlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        //创建一个容量为3的阻塞队列
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        System.out.println(blockingQueue.offer("d"));
    }
}
```

结果

```shell
true
true
true
false
```



> poll 移除 同样 移除的元素不存在也不会报错，而是返回null

```Java
public class BlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        //创建一个容量为3的阻塞队列
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(blockingQueue.offer("a"));
        System.out.println(blockingQueue.offer("b"));
        System.out.println(blockingQueue.offer("c"));
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
    }
}
```

结果

```shell
true
true
true
a
b
c
null
```



> peek() 和element()一样，不再演示



#### 阻塞代码演示

> put 插入，没有返回值，当发生阻塞无法插入时会一直等待pull或线程退出

```Java
public class BlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        //创建一个容量为3的阻塞队列
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
        blockingQueue.put("a");
        blockingQueue.put("b");
        blockingQueue.put("c");
        blockingQueue.put("d");
    }
}
```

结果

![image-20200818145712257](http://picture.youyouluming.cn/image-20200818145712257.png)



> take 移除元素，当阻塞队列没有元素时会一直等待

```Java
public class BlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        //创建一个容量为3的阻塞队列
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
        blockingQueue.put("a");
        blockingQueue.put("b");
        blockingQueue.put("c");

        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
    }
}
```

结果

![image-20200818145853295](http://picture.youyouluming.cn/image-20200818145853295.png)

#### 超时代码演示

> offer 插入，指定超时时间，到期后自动终止

```Java
public class BlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        //创建一个容量为3的阻塞队列
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(blockingQueue.offer("a", 3, TimeUnit.SECONDS));
        System.out.println(blockingQueue.offer("b", 3, TimeUnit.SECONDS));
        System.out.println(blockingQueue.offer("c", 3, TimeUnit.SECONDS));
        System.out.println(blockingQueue.offer("d", 3, TimeUnit.SECONDS));
    }
}
```

![image-20200818151417811](http://picture.youyouluming.cn/image-20200818151417811.png)



> poll 移除  指定超时时间，过期自动终止

```Java
public class BlockingQueueDemo {
    public static void main(String[] args) throws InterruptedException {
        //创建一个容量为3的阻塞队列
        BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(blockingQueue.offer("a", 3000, TimeUnit.SECONDS));
        System.out.println(blockingQueue.offer("b", 3000, TimeUnit.SECONDS));
        System.out.println(blockingQueue.offer("c", 3000, TimeUnit.SECONDS));
        System.out.println(blockingQueue.poll(3, TimeUnit.SECONDS));
        System.out.println(blockingQueue.poll(3, TimeUnit.SECONDS));
        System.out.println(blockingQueue.poll(3, TimeUnit.SECONDS));
        System.out.println(blockingQueue.poll(3, TimeUnit.SECONDS));
    }
}
```

结果

![image-20200818152440451](http://picture.youyouluming.cn/image-20200818152440451.png)

## 三、SynchronousQueue

使用同步队列实现新版的消费者生产者

> 新版消费者生产者

```java
public class BlockingQueueDemo01 {
    public static void main(String[] args){
        BlockingQueue<String> blockingQueue = new SynchronousQueue<>();

        new Thread(() -> {
            try {
                System.out.println(Thread.currentThread().getName() + "线程执行put-a");
                blockingQueue.put("a");
                System.out.println(Thread.currentThread().getName() + "线程执行put-b");
                blockingQueue.put("b");
                System.out.println(Thread.currentThread().getName() + "线程执行put-c");
                blockingQueue.put("c");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"A").start();

        new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName() + "线程取出的值：" + blockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName() + "线程取出的值：" + blockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName() + "线程取出的值：" + blockingQueue.take());

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"B").start();
    }
}
```

结果：

![image-20200822114833375](http://picture.youyouluming.cn/image-20200822114833375.png)

**说明：**

`SynchronousQueue`只会保存一个数据，只有当前数据被取出才会保存第二个数据。