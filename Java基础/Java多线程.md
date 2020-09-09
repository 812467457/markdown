# 多线程

## 一、概述

进程是程序的一次执行过程。

线程是一个程序内部的一条执行路径，若一个进程同一时间并行执行多个线程，就是支持多线程。所谓多线程是指一个进程在执行过程中可以产生多个更小的程序单元，这些更小的单元称为线程，这些线程可以同时存在，同时运行，一个进程可能包含多个同时执行的线程。



## 二、创建线程

### 1、方式一：继承Thread类

* 创建一个继承与Thread类的子类
* 重写run方法，此方法执行具体操作
* 调用该类的start方法启动线程

```Java
class MyThread extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0) {
                System.out.println(i);
            }
        }
    }
}

public class Thread01 {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        //自定义的线程，调用此对象的start方法：启动线程、调用当前线程的run方法
        myThread.start();
        //主线程
        for (int i = 0; i < 10000; i++) {
            if (i % 2 != 0) {
                System.out.println(i);
            }
        }
    }
}
```



###  2、方式二：实现Runnable接口

* 创建应该实现Runnable接口的类
* 实现接口中的抽象方法Run()

* 创建这个实现类的对象，作为参数传入Thread类的构造器，作为一个线程
* start启动线程

```Java
class MyThread02 implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0) {
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
        }
    }
}

public class Thread02 {
    public static void main(String[] args) {
        MyThread02 myThread02 = new MyThread02();
        Thread thread = new Thread(myThread02);
        Thread thread2 = new Thread(myThread02);
        thread.start();
        thread2.start();

    }
}
```



## 三、线程常用方法

* start()：启动当前线程，调用当前线程的run()
* run()：run方法中写这个线程具体做什么
* currentThread()：返回当前执行的线程
* getName()：返回当前线程名字
* setName()：指定当前线程名字
* yield()；释放当前cpu执行权
* join()：当前线程进入阻塞状态，等其他线程执行完毕再次接着执行
* stop()：强制结束当前线程
* sleep()：休眠当前线程，需要指定休眠时间
* isAlive()：判断当前线程是否还存活



## 四、线程的优先级

### 1、线程的调度

#### 1.1、调度策略

* 时间片：每个进程被分配一时间段，即该进程允许运行的时间。
* 抢占式：高优先级的线程抢占cpu资源

#### 1.2、Java调度方法

* 同优先级线程使用时间片策略
* 对高优先级线程使用抢占策略

#### 1.3、线程优先等级

* 最大优先级 MAX_PRIORITY：10
* 最小优先级 MIN_PRIORITY：1
* 默认优先级 NORM_PRIORITY：5

#### 1.4、优先级相关方法

* getPriority()：返回线程优先级
* setPriority(int newPriority)：设置线程优先级



## 五、线程的生命周期

* 新建：当一个Thread类或其子类被声明创建时，线程处于新建状态。
* 就绪：处于新建状态的线程被start后，进入线程队列等待CPU时间片，此时具备运行条件，但没分配到cpu资源
* 运行：当就绪线程被调度并获得cpu资源时，便进入运行状态
* 阻塞：在某种情况下，被人为的挂起或者执行输入输出操作时，让出cpu执行器并临时终止运行。
* 死亡：线程完成它全部的工作或线程被提前强制性的终止或出现异常导致结束。

![image-20200815180509210](http://picture.youyouluming.cn/image-20200815180509210.png)

## 六、线程的同步

### 1、线程安全问题

在多个线程操作同一个共享的数据时，看你会造成数据的不完整性，破坏数据。比如两个售票窗口同时卖票，在最后还剩一张票时，两个窗口同时进入了判断都为true，那么就会发生超卖或重复卖的现象。

```Java
class MyThread02 implements Runnable {
    private int ticket = 1000;
    @Override
    public void run() {
        while (true){
            if (ticket > 0) {
                System.out.println(Thread.currentThread().getName() + ":" + ticket--);
            } else {
                break;
            }
        }
    }
}

public class Thread02 {
    public static void main(String[] args) {
        MyThread02 myThread02 = new MyThread02();
        Thread thread = new Thread(myThread02);
        Thread thread2 = new Thread(myThread02);
        thread.setName("窗口1");
        thread2.setName("窗口2");
        thread.start();
        thread2.start();

    }
}
```



#### 1.1、解决方式一：同步代码块

* 操作共享数据的代码即为需要同步的代码
* 需要一个监视器，也就是锁，任何一个类的对象都可以做锁 ，多个线程共用同一个锁
* 同步代码块时，只有一个线程可以操作，其他线程需要等待上一个线程操作完后去抢cpu执行权

```Java
class MyThread02 implements Runnable {
    private int ticket = 100;
    @Override
    public void run() {
        while (true){
            synchronized (this) {
                if (ticket > 0) {
                    try {
                        Thread.sleep(300);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + ":" + ticket--);
                } else {
                    break;
                }
            }
        }
    }
}
```

#### 1.2、解决方式二：同步方法

* 把使用到共享数据的代码提取出来一个单独的方法
* 然后在方法上加synchronized表示该方法为同步方法

```Java
class MyThread02 implements Runnable {
    private int ticket = 100;

    @Override
    public void run() {
        while (true) {
            if (!show()) {
                break;
            }
        }
    }

    public synchronized boolean show() {
        if (ticket > 0) {
            System.out.println(Thread.currentThread().getName() + ":" + ticket--);
            return true;
        } else {
            return false;
        }
    }
}
```

#### 1.3、解决方式三：Lock锁

* Java5.0开始，提供了更强大的线程同步机制，通过显式定义同步锁对象来实现同步。使用Lock对象做锁
* Lock接口是控制多个线程对共享数据资源进行访问的工具，每次只能有一个线程对lock加锁，线程访问共享资源之前要先获得lock对象
* ReentrantLock类实现了Lock接口，它和synchronized相同的并发性和内存语义
* lock可以指定只锁一小部分使用到共享资源的代码，相比synchronized效率更高

```Java
class MyThread03 implements Runnable {
    private int ticket = 100;
    //ReentrantLock(是否使用公平策略)
    private ReentrantLock lock = new ReentrantLock(true);

    @Override
    public void run() {
        while (true) {
            try {
                //加锁
                lock.lock();
                Thread.sleep(50);
                if (ticket > 0) {
                    System.out.println(Thread.currentThread().getName() + ":" + ticket--);
                } else {
                    break;
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                //解锁
                lock.unlock();
            }
        }
    }
}
```

* synchronized和lock的区别
  * 相同：二者都可以解决线程安全的问题
  * 不同：synchronized执行完同步代码后自动释放同步监视器，lock需要手动启动和结束同步。

### 2、死锁问题

* 不同的线程分别占用对方所需要的同步资源不放弃，都在等对方放弃自己需要的同步资源，形成死锁。
* 出现死锁后不会出现异常或提示，但是程序都处于阻塞状态，不能继续运行。

死锁演示

```Java
public class Thread03 {
    public static void main(String[] args) {
        StringBuffer sb1 = new StringBuffer();
        StringBuffer sb2 = new StringBuffer();

        new Thread() {
            @Override
            public void run() {
                synchronized (sb1) {
                    sb1.append("a");
                    sb2.append("1");
                    synchronized (sb2) {
                        sb1.append("b");
                        sb2.append("2");
                        System.out.println(sb1);
                        System.out.println(sb2);
                    }
                }
            }
        }.start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (sb2) {
                    sb1.append("c");
                    sb2.append("3");
                    synchronized (sb1) {
                        sb1.append("d");
                        sb2.append("4");
                        System.out.println(sb1);
                        System.out.println(sb2);
                    }
                }
            }
        }).start();
    }
}
```

如果第一个线程先执行，把sb1加速进入操作的同时第二个线程也执行了，把sb2加锁进入操作，这时第一个线程想继续往下走就需要sb2，但是sb2在第二个线程中被加锁了，并且sb2也需要sb1才能继续往下执行，双方处于一个互相等待对方释放自己所需资源的状态。



## 七、线程的通信

### 1、线程通信的方法

* wait()：让当前线程进入阻塞状态，并释放同步监视器
* notifyAll()/notify()：唤醒其他所有wait()线程/唤醒一个线程
* 这三个方法不是线程的方法，是Object类的方法

* 这些方法必须使用在同步代码块或同步方法（synchronized）中

```Java
class MyThread04 implements Runnable{
    private int num = 1;
    
    @Override
    public void run() {
        while (true) {
            synchronized (this) {
                notify();
                if (num <= 100) {
                    System.out.println(Thread.currentThread().getName() + ":" + num);
                    num++;

                    try {
                        wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                } else {
                    break;
                }
            }
        }
    }
}
```

* sleep()和wait()方法的异同
  * 相同：一旦执行方法，都会使线程进入阻塞状态
  * 不同：①两个方法声明位置不同，Thread类中声明sleep()，Object类声明wait()。②调用要求不同，sleep可以在任何场景下调用，wait必须在同步代码下调用。③都在同步代码中使用的情况，sleep不会释放锁，wait会释放锁



### 2、生产者消费者

```Java
class Producer{
    private int num = 0;
    public synchronized void add() throws InterruptedException {
        while (num != 0) {
            this.wait();
        }
        num++;
        System.out.println(Thread.currentThread().getName() + ":" + num);
        this.notify();
    }

    public synchronized void div() throws InterruptedException {
        while (num == 0) {
            this.wait();
        }
        num--;
        System.out.println(Thread.currentThread().getName() + ":" + num);
        this.notify();
    }
}


public class ConsumerAndProducer {
    public static void main(String[] args) {
        MyNum Producer = new Producer();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    myNum.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    myNum.div();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    myNum.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    myNum.div();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"D").start();
    }
}
```

注意：在多线程数据交互判断时，必须使用while做判断，如果使用if做判断，当存在多个消费者时，如果A线程在wait，这时C线程也来到同一个方法中并且进行完了判断，这时A线程被notify唤醒处理完数据，接着C线程也会进来再处理一遍数据，这时数据就会错乱。使用while判断时，虚假唤醒会进行重新判断。

## 八、新的线程创建方式

### 1、方式三：实现Callable接口

和Runnable接口相比，Callable更强大：

* 相比run()方法，call()方法可以有返回值
* call()方法可以抛出异常
* 支持泛型的返回值
* 借助FutureTask类获取返回结果



```Java
class MyThread05 implements Callable<Object> {
    private int sum = 0;
    @Override
    public Object call() throws Exception {
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0) {
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
            sum += i;
        }
        return sum;
    }
}

public class Thread06 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyThread05 myThread05 = new MyThread05();
        FutureTask<Object> futureTask = new FutureTask<>(myThread05);
        new Thread(futureTask).start();

        //获取call方法返回值
        Object o = futureTask.get();
        System.out.println(o);
    }
}
```

为什么使用Callable？

* 可以针对返回值对线程进行更加细粒度化的控制





### 2、方式四：使用线程池

好处：

* 提高响应速度
* 降低资源销毁
* 便于线程管理 
  * corePoolSize：核心池大小
  * maximumPoolSize：最大线程
  * keepAliveTime：线程没有任务多长时间终止

创建方法：

* jdk5起提供了线程池相关API：ExecutorService和Executors类
* ExecutorService：线程池接口，常见子类ThreadPoolExecutor
* Executors：工具类、线程池工厂类，用于创建并返回不同类型的线程池
  * Executors.newCachedThreadPool()：创建一个根据需要的线程池
  * Executors.newFixedThreadPool(n)：创建一个固定数量的线程池
  * Executors.newSingleThreadExecutor()：创建一个只有一个线程的线程池
  * Executors.newSingleThreadExecutor(n)：创建一个线程池，可以安排延后或定期运行。



```Java
class MyThread07 implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0) {
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
        }
    }
}

class MyThread08 implements Callable<Object> {

    @Override
    public Object call() throws Exception {
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0) {
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
        }
        return null;
    }
}

public class Thread07 {
    public static void main(String[] args) {
        //创建指定数量的线程池
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        //设置线程属性
        ThreadPoolExecutor executor = (ThreadPoolExecutor) executorService;
        executor.setCorePoolSize(10);
        //适用于Runnable
        executorService.execute(new MyThread07());
        //适用于Callable
        executorService.submit(new MyThread08());
        //关闭线程池
        executorService.shutdown();
    }
}
```





