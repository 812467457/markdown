# Java面试之锁

## 一、公平锁和非公平锁

### 1、是什么？

平时用的`synchronized`和`ReentrantLock（默认情况下参数是false）`都是非公平锁，`ReentrantLock(true)`是公平锁

公平锁是指多个线程按照申请锁的顺序来获取锁。（默认情况是非公平）

非公平锁指后来的线程可能比前面的线程先获取锁，在高并发下有可能会造成优先级反转和饥饿现象。

### 2、区别

* 公平锁：在并发环境中，每个线程在获取锁时会先查看此锁维护的等待队列，如果为空，或者当前线程时第一个，就占有锁，否则就会加入到等待队列中，按照先进先出的规则从队列中取到自己。
* 非公平锁：上来就尝试占用锁，如果尝试失败，就采用公平锁的方式



## 二、可重入锁（递归锁）

### 1、是什么？

`synchronized`和`ReentrantLock`就是一个典型的可重入锁。

可重入锁指的是同一线程外层函数获取锁后，内层递归函数仍然能获得该锁的代码。

在同一个线程在外层方法获取锁后，在进入内层方法会自动获取锁，线程可以进入任何一个它已经拥有的锁所同步着的代码块。

**可重入锁最大作用就是避免死锁**



### 2、代码演示

> synchronized版重入锁

```Java
class MyData{
    public synchronized void method1(){
        System.out.println(Thread.currentThread().getName() + ":method1");
        method2();
    }
    public synchronized void method2(){
        System.out.println(Thread.currentThread().getName() + ":method2");
    }
}

public class ReentrantLockDemo01 {
    public static void main(String[] args) {
        MyData myData = new MyData();
        new Thread(()->{
            myData.method1();
        },"A").start();
    }
}
```

结果

![image-20200821143050996](http://picture.youyouluming.cn/image-20200821143050996.png)

**说明：**

资源类中有method1和method2两个同步方法，其中method1还调用了method2方法，一个线程A去调用method1方法时也会调用到method2方法，因为method2在method1内部的调用可以无视method锁，并且两个方法用的是同一把锁。



> ReentrantLock版重入锁

```Java
class MyData{
    Lock lock = new ReentrantLock();
    public void method1(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + ":method1");
            method2();
        } finally {
            lock.unlock();
        }
    }

    public void method2(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + ":method2");
        } finally {
            lock.unlock();
        }
    }
}

public class ReentrantLockDemo01 {
    public static void main(String[] args) {
        MyData myData = new MyData();
        new Thread(()->{
            myData.method1();
        },"A").start();
    }
}
```

原理同上，需要注意一点，如果多次加锁（lock.lock()）,必须也有对应多个解锁(lock.unlock)才能正常运行。



## 三、自旋锁

### 1、是什么

自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗cpu

### 2、手写一个自旋锁

> 自旋锁代码

```java
public class ReentrantLockDemo02 {
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    public void myLock() {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + "线程进入mylock");

        while (!atomicReference.compareAndSet(null, thread)) {

        }
    }

    public void myUnLock() {
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread, null);
        System.out.println(thread.getName() + "线程myUnLock");
    }


    public static void main(String[] args) {
        ReentrantLockDemo02 reentrantLockDemo02 = new ReentrantLockDemo02();
        new Thread(() -> {
            reentrantLockDemo02.myLock();
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            reentrantLockDemo02.myUnLock();
        }, "A").start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            reentrantLockDemo02.myLock();
            reentrantLockDemo02.myUnLock();
        }, "B").start();
    }
}
```

结果：

![image-20200821202139103](http://picture.youyouluming.cn/image-20200821202139103.png)

**说明：**

使用原子引用线程，默认没有值为null，资源类中有两个方法，一个加锁一个解锁，其中加锁的设置一个while循环，判断当前compareAndSet是否设置成功，如果设置成功就跳出循环，没有设置成功一直循环直到成功。创建A、B两个线程，先让A线程访问加锁的方法并且设置成功，现在atomicReference中的值就是Thread对象，并且睡眠5秒，此时线程B也会访问到加锁的方法，但是atomicReference已经被A修改了，B的while循环会一直循环下去直到A被唤醒走解锁的方法，B才会判断成功跳出循环并解锁。



## 四、读写锁（互斥锁）



### 1、是什么？

* 独占锁（写锁）：指该锁一次只能被一个线程所持有。`synchronized`和`ReentrantLock`都是独占锁。
* 共享锁（读锁）：指该锁可以被多个线程所持有。`ReentrantReadWriteLock`其读锁就是共享锁，其写锁就是独占锁。读锁的共享可以保证并发读是非常高效的，读写，写读，写写的过程是互斥的。



代码可以参考之前的笔记：https://juejin.im/post/6862616894500061191