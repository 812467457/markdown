# CAS（比较交换）

## 一、CAS是什么？

CAS是java.util.concurrent.atomic包下AtomicInteger类的compareAndSet方法，该方法的作用是比较并交互。

代码演示：

```Java
public class CASDemo {
    public static void main(String[] args) {
        //创建一个整型的原子类，指定值为5
        AtomicInteger atomicInteger = new AtomicInteger(5);
        //比较并交互CAS,compareAndSet(期望值，更新值)
        System.out.println(atomicInteger.compareAndSet(5, 10) + ":" + Thread.currentThread().getName() + ":" + atomicInteger.get());
        System.out.println(atomicInteger.compareAndSet(5, 20) + ":" + Thread.currentThread().getName() + ":" + atomicInteger.get());
    }
}
```

结果：

![image-20200820105220481](http://picture.youyouluming.cn/image-20200820105220481.png)

**说明：**

`AtomicInteger atomicInteger = new AtomicInteger(5)`相当于在主内存存储了一个变量`atomicInteger `，值为5，`atomicInteger.compareAndSet(5, 10)`给定一个期望值，和一个更新值。期望值表示等到更新完写回主内存时，期望主内存中的值是多少。更新值就是更新后的数据。如果执行成功返回true，如果期望值和主内存值不同，返回false。



## 二、CAS底层原理是什么？

### 1、unsafe加自旋锁

CAS源码

![image-20200820112408447](http://picture.youyouluming.cn/image-20200820112408447.png)

unsafe是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地方法（native）来访问，unsafe相当于一个后门，基于该类可以直接操作特定内存数据。unsafe在sun,misc包。

注意：unsafe类中所有方法都是native修饰的，表示unsafe类中的方法都是操作系统底层资源执行相应任务。

valueOffset表示该变量在内存中的偏移地址，因为unsafe就是根据内存偏移地址获取数据的

### 2、从底层看CAS是什么

它是一条cpu并发原语，功能是判断内存某个位置的值是否为预期值，如果是则改为更新值，不是就不做操作，这个过程就是原子性。



## 三、CAS缺点是什么？

* 循环时间长，cpu开销大
* 只能保证一个共享变量的原子操作
* 引出ABA问题



# ABA问题

## 1、ABA问题是什么？

CAS导致到最大的问题ABA。

CAS算法实现的一个重要前提是，取出内存中某时某刻的数据并在当下时刻比较替换，那么这个时间差会导致数据的变化。

比如线程A从主内存中取出数据1，这时线程B也取出数据1，并且线程B进行一系列的操作把数据从1变为2，然后线程B又把主内存的数据变回1，这时线程A进行CAS操作发现主内存的数据仍然是1，然后线程A操作成功。

虽然最后线程的CAS操作都成功了，但是不代表这个过程就是没有问题的。



### 2、原子引用

之前使用过`AtomicInteger`保证了整数类型的数据原子性，但是如果是自定义类型的怎么保证数据的原子性呢？这里就需要使用到原子引用了`AtomicReference`。

> 代码演示

```Java
@Data
@AllArgsConstructor
@ToString
class User {
    private String name;
    private int age;
}

public class ABADemo {
    public static void main(String[] args) {
        User user1 = new User("jack", 11);
        User user2 = new User("marry", 12);
        AtomicReference<User> atomicReference = new AtomicReference<>();
        atomicReference.set(user1);
        System.out.println(atomicReference.compareAndSet(user1, user2) + ":" + atomicReference.get().toString());
        System.out.println(atomicReference.compareAndSet(user1, user2) + ":" + atomicReference.get().toString());
    }
}
```

结果

![image-20200820163233207](http://picture.youyouluming.cn/image-20200820163233207.png)

**说明：**

对引用类型进行原子包装，同样保证了其原子性，每次从内存中修改数据都需要和预期值对比，符合就修改成功，不符合就失败。同样存在ABA问题

### 3、解决ABA问题-时间戳原子引用

时间戳原子引用就是相当于在之前的基础加一个版本号，每一次修改都会版本号加一，这样就可以防止出现ABA问题了，使用`AtomicStampedReference`类



> ABA问题代码演示，未使用时间戳原子引用

```java
public class ABADemo {
    public static void main(String[] args) {
        AtomicReference<Integer> atomicReference = new AtomicReference<>(100);

        new Thread(() -> {
            atomicReference.compareAndSet(100, 101);
            atomicReference.compareAndSet(101, 100);
        }, "A").start();
        new Thread(() -> {
             try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(atomicReference.compareAndSet(100, 200) + ":" + atomicReference.get());
        }, "B").start();
    }
}
```

结果：

![image-20200820172034342](http://picture.youyouluming.cn/image-20200820172034342.png)

**说明：**

A线程先获取主内存的atomicReference变量的值，然后进行CAS操作，把100换为101，第一次操作完，再次进行操作，把101换为100，这时线程B开始操作，线程B预期100更新值200，然后主内存中的值也是100所以线程B操作成功，但是这个值已经被线程A修改过了，需要避免这种情况。



> 使用了时间戳原子引用代码

```java
public class ABADemo {
    public static void main(String[] args) {
        // AtomicStampedReference<>(初始值, 初始版本号)
        AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(100, 1);

        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + ":第一次版本号:" + stamp + ",值为：" + atomicStampedReference.getReference());
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //compareAndSet(期望值，更新值，当前版本号，跟新版本号)
            atomicStampedReference.compareAndSet(
                    100,
                    101,
                    atomicStampedReference.getStamp(),
                    stamp += 1);

            System.out.println(Thread.currentThread().getName() + ":第二次版本号:" + atomicStampedReference.getStamp() + ",值为：" + atomicStampedReference.getReference());

            atomicStampedReference.compareAndSet(
                    101,
                    100,
                    atomicStampedReference.getStamp(),
                    stamp += 1);

            System.out.println(Thread.currentThread().getName() + ":第三次版本号:" + atomicStampedReference.getStamp() + ",值为：" + atomicStampedReference.getReference());
        }, "A").start();
        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + ":第一次版本号:" + atomicStampedReference.getStamp());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("运行结果：" + atomicStampedReference.compareAndSet(
                    100,
                    200,
                    stamp,
                    stamp + 1));
            System.out.println(Thread.currentThread().getName() + ":当前版本号:" + atomicStampedReference.getStamp());
            System.out.println(Thread.currentThread().getName() + ":当前最新值:" + atomicStampedReference.getReference());
        }, "B").start();
    }
}
```

结果：

![image-20200820205949242](http://picture.youyouluming.cn/image-20200820205949242.png)

**说明：**

依然是A、B两个线程，创建一个时间戳原子类对象，给定值为100，版本号为1，两个线程前面都有一段sleep为了保证双方都能获得初始的版本号，线程A获得执行权，对atomicStampedReference进行ABA操作，并且每次操作都会让版本号+1，等A线程执行完，此时atomicStampedReference中的值为100，版本号为3，来到B线程，B线程获取的版本号还是1，B线程就拿着期望值100和版本号1去对atomicStampedReference进行修改，这一次虽然期望值对上了，但是版本号对不上然后操作失败。

