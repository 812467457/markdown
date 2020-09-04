# volatile

## 一、请谈谈你对Volatile的理解

* volatile是Java虚拟机提供的轻量级的同步机制，三大特性如下
  * 保证可见性
  * 不保证原子性
  * 禁止指令重排

## 二、什么是保证可见性？

在回答这个问题前需要先了解JMM相关知识

### 1、谈谈什么是JMM？

JVM是Java虚拟机。

JMM是Java内存模型。

JMM是一种抽象的概念并不真实存在，它描述的是一组规范或规则。

JMM相关规定：

* 线程解锁前，必须把共享变量的值刷回主内存（可见性，保证某个线程对资源修改后通知其他线程）
* 线程加锁前，必须读取主内存最新的值到自己的工作内存
* 加锁解锁是同一把锁

JMM三大特性：

* 可见性 
* 原子性
* 有序性

### 2、可见性

运行程序的实体是线程，而每个线程创建时都会被分配一个私人的工作空间，但是Java中所有的变量都存储在主内存，主内存共享给其他所有线程访问，线程如果对变量进行操作，必须先把这个变量复制到自己的工作内存空间，操作完成再把这个变量写回主内存，线程不能直接操作主内存的变量。如果这个时候存在多个线程都去操作主线程中的变量，不同的线程之间无法访问对方的工作内存空间，线程间的通信必须通过内存来完成，这就是可见性的含义。

访问过程图：

![image-20200819180741177](http://picture.youyouluming.cn/image-20200819180741177.png)

代码演示：

>  没有使用volatile关键字，没有可见性

```Java
class MyData {
    int num = 0;

    public void addNum() {
        this.num = 60;
    }
}

public class VolatileDemo01 {
    public static void main(String[] args) {
        MyData myData = new MyData();
        new Thread(() -> {
            try {
                System.out.println(Thread.currentThread().getName() + ":执行");
                TimeUnit.SECONDS.sleep(3);
                myData.addNum();
                System.out.println(Thread.currentThread().getName() + ":执行完毕，改变后的值为" + myData.num);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"A").start();

        while (myData.num == 0) {

        }
        System.out.println("执行完毕，num的值为" + myData.num);
    }
}
```

结果

![image-20200819195605404](http://picture.youyouluming.cn/image-20200819195605404.png)

**说明：**

有一个MyData资源类，有一个变量num初始值为0和一个改变num变量的方法addNum，在main方法创建一个线程A，A开始执行先睡眠三秒，然后去调用addNum改变了资源类中的变量num为60并写回内存，与此同时，main方法也有一个线程一直在循环中直到num的值不为0，可是num的值已经通过A线程改变了，但main线程仍然不知情，依旧在循环中，这就是没有可见性代码的演示。



> 使用volatile后的结果

```java 
class MyData {
    volatile int num = 0;

    public void addNum() {
        this.num = 60;
    }
}
```

结果：

![image-20200819201251752](http://picture.youyouluming.cn/image-20200819201251752.png)

**说明：**

可以发现，main线程已经可以正常接收到通知了。使用volatile后，如果哪个线程对主内存的变量做了修改，就会立马通知其他线程这个变量已经被修改，并且重新从主内存获取最新的值。



**主要考点：**

1. 主内存的变量不是直接进行修改的，而是每个线程都有一个私人的内存工作空间，线程会先把主内存的变量拷贝一份到自己的内存工作空间，然后进行操作，操作完在重新写回主内存。
2. 可见性，当某个线程对主内存的变量进行了操作，要及时通知其他线程做更新操作，保持和主内存的数据一致。



## 三、原子性

原子性：表示某个操作不可分割，保证数据完整一致性，要么同时成功，要么同时失败。

### 1、volatile不保证原子性验证

代码演示：

```Java
class MyData2 {
    volatile int num = 0;

    public void addPlus() {
        num++;
    }
}

public class VolatileDemo02 {
    public static void main(String[] args) throws InterruptedException {
        MyData2 myData2 = new MyData2();

        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    myData2.addPlus();
                }
            }).start();
        }

        //设置main线程先等待上面的线程执行完再执行，Thread.activeCount()获取当前线程数
        while (Thread.activeCount() > 2) {
            //线程礼让，让其他线程先执行
            Thread.yield();
        }
        System.out.println(myData2.num);
    }
}
```

![image-20200819204423134](http://picture.youyouluming.cn/image-20200819204423134.png)

**说明：**

创建20个线程，对资源类中的变量num（初始值为0）进行++操作，每个线程循环1000次，按照预期结果最后num=20*1000=20000才对，可结果并不如愿，说明了volatile不保证原子性。



### 2、volatile为什么不保证原子性

之前已经说过每个线程都有一个私人的内存工作空间，并且这个add方法没有加synchronized（锁），所以会有多个线程去争抢同一个资源的现象，虽然说volatile具有可见性，能够通知其他线程主内存的数据变化，但是在多个线程争抢中整个过程发生的太快，很有可能通知的时候已经写回到内存了，这样就会出现漏写的情况



### 3、volatile不保证原子性的问题解决

第一种方法：加synchronized，但是synchronized弊端过多，比如：性能损耗、产生阻塞，所以换一种更好的方式。

**第二种方法：使用Java.util.coucurrent.atomic（JUC.原子）包下的AtomicInteger（原子整型包装类）**

**代码演示：**

```Java
class MyData3{
    //创建一个原子整型类，默认值为0
    AtomicInteger atomicInteger = new AtomicInteger();
    public void addAtomic(){
        //每次加1等同于atomicInteger++
        atomicInteger.getAndIncrement();
    }
}

public class VolatileDemo03 {
    public static void main(String[] args) {
        MyData3 myData3 = new MyData3();

        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    myData3.addAtomic();
                }
            }, "第" + i + "线程").start();
        }

        //设置main线程先等待上面的线程执行完再执行，Thread.activeCount()获取当前线程数
        while (Thread.activeCount() > 2) {
            //线程礼让，让其他线程先执行
            Thread.yield();
        }
        System.out.println(myData3.atomicInteger);
    }
}
```

结果：

![image-20200819220159940](http://picture.youyouluming.cn/image-20200819220159940.png)

为什么使用AtomicInteger可以做到保证原子性，getAndIncrement最终调用的也是CAS，请参考CAS相关笔记。



## 四、指令重排

### 1、指令重排是什么？

计算机在执行我们写的代码时，为了提高性能，编译器和处理器通常会对指令重排，也就是说我们写代码的顺序不代表编译器编译的顺序。在**单线程**情况下执行代码的顺序一致。处理器在进行重排时必须考虑指令之间的**数据依赖性**，但是在**多线程**交替执行的情况下，由于编译器优化重排的存在，两个线程使用的变量能否保持一致无法确定，结果无法预测。

### 2、案例

![image-20200820000708634](http://picture.youyouluming.cn/image-20200820000708634.png)

在多线程环境下去调用整个资源类，按照我们预先的执行顺序和结果应该是：语句1=>语句2=>语句3，最后输出为6。但是因为有指令重排的存在，执行顺序就有几率变为：语句2=>被第二个线程抢到执行权=>语句3=又被第一个线程抢到=>语句1，最后输出为1.

### 4、底层原理（了解）

![image-20200820001325608](http://picture.youyouluming.cn/image-20200820001325608.png)

![image-20200820001350837](http://picture.youyouluming.cn/image-20200820001350837.png)

## 五、volatile使用总结

* 对于工作内存和主内存同步延迟现象导致的可见性问题：可以使用synchronized或volatile关键字解决，它们都可以使一个线程修改后的变量立即对其他线程可见。
* 对于指令重排导致的可见性问题和有序性问题：可以使用volatile关键字解决，因为volatile的另一个作用就是禁止指令重排

## 六、单例模式volatile

### 1、多线程下单例模式出现的问题

> 一个正常的单例模式

```Java
class SingletonDemo{
    private static SingletonDemo instance = null;

    private SingletonDemo() {
        System.out.println(Thread.currentThread().getName() + ":构造方法");
    }

    public static SingletonDemo getInstance(){
        if (instance == null) {
            instance = new SingletonDemo();
        }
        return instance;
    }
}

public class VolatileDemo04 {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                SingletonDemo.getInstance();
            },i + "").start();
        }
    }
}
```

结果：

![image-20200820003249821](http://picture.youyouluming.cn/image-20200820003249821.png)

可以发现单例模式被创建了三次对象。



### 2、解决问题

当然可以使用直接在方法上加synchronized关键字来解决，但没必要，synchronized会把整个方法锁上，影响效率。

**使用DCL版单例模式（双端检锁机制）**

> 使用同步代码块方式

```Java
class SingletonDemo{
    private static SingletonDemo instance = null;
    
    private SingletonDemo() {
        System.out.println(Thread.currentThread().getName() + ":构造方法");
    }

    public static SingletonDemo getInstance(){
        if (instance == null) {
            synchronized (SingletonDemo.class) {
                if (instance == null) {
                    instance = new SingletonDemo();
                }
            }
        }
        return instance;
    }
}

public class VolatileDemo04 {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                SingletonDemo.getInstance();
            },i + "").start();
        }
    }
}
```

结果：

![image-20200820004457841](http://picture.youyouluming.cn/image-20200820004457841.png)

说明：双端加锁，在加锁前和加锁后都会进行一次判断。

**注意：双端检锁机制不一定线程安全，虽然同步代码块的双端加锁看起来似乎解决了多线程下单例模式的安全问题，但是因为有指令重排的机制存在，会有小概率出现异常。**

**原因：在初始化对象时并不是马上就能初始化完成，这中间有一定的过程，但是这个时候可能已经分配了引用和内存地址，但是instance=null，因为实例化这个对象和接下来对内存中的instance赋值没有直接依赖，所以指令重排可能会直接把这个等于null的instance返回，就造成了线程安全问题。如下图代码所示**

![image-20200820010150037](http://picture.youyouluming.cn/image-20200820010150037.png)



> 使用同步代码块加volatile方式改进

```Java
class SingletonDemo{
    private volatile static SingletonDemo instance = null;

    private SingletonDemo() {
        System.out.println(Thread.currentThread().getName() + ":构造方法");
    }

    public static SingletonDemo getInstance(){
        if (instance == null) {
            synchronized (SingletonDemo.class) {
                if (instance == null) {
                    instance = new SingletonDemo();
                }
            }
        }
        return instance;
    }
}

public class VolatileDemo04 {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                SingletonDemo.getInstance();
            },i + "").start();
        }
    }
}
```

