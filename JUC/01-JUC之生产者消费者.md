# JUC之生产者消费者

模拟一个情况，现在有两个线程对一个资源进行操作，资源数据默认为0，一个线程做减法操作，另一个线程做加法操作，每个线程进行十轮操作，保证最后资源的数据依然是0。

代码如下：

```Java
class Mymethod{
    private int num = 0;
    public synchronized void add() throws InterruptedException {
        if (num != 0) {
            this.wait();
        }
        num++;
        System.out.println(Thread.currentThread().getName() + ":" + num);
        this.notify();
    }

    public synchronized void div() throws InterruptedException {
        if (num == 0) {
            this.wait();
        }
        num--;
        System.out.println(Thread.currentThread().getName() + ":" + num);
        this.notify();
    }
}

public class ConsumerAndProducer {
    public static void main(String[] args) {
        Mymethod mymethod = new Mymethod();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    mymethod.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    mymethod.div();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"B").start();
    }
}
```

结果如下

![image-20200816234704762](http://picture.youyouluming.cn/image-20200816234704762.png)

使用lambda表达式和匿名内部类的方式创建两个线程（A,B）并且启动，A线程去调用资源中加操作，B线程去调用资源中的减操作。假如A线程抢到执行权进入判断此时num等于0，所以判断为否，执行num++操作，任何进行第二轮的争抢，假如又是A线程抢到了执行权，这次判断为是，A线程进入等待，B线程获得执行权，然后B线程执行num--操作，并且notify把线程A放开，如此循环直到最后num依然是0。

但是，这时候又加入了两线程C和D，同样一个执行加操作一个执行减操作，那么情况又会是怎样的呢？

代码：

```Java
class Mymethod{
    private int num = 0;
    public synchronized void add() throws InterruptedException {
        if (num != 0) {
            this.wait();
        }
        num++;
        System.out.println(Thread.currentThread().getName() + ":" + num);
        this.notifyAll();
    }

    public synchronized void div() throws InterruptedException {
        if (num == 0) {
            this.wait();
        }
        num--;
        System.out.println(Thread.currentThread().getName() + ":" + num);
        this.notifyAll();
    }
}
public class ConsumerAndProducer {
    public static void main(String[] args) {
        Mymethod mymethod = new Mymethod();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    mymethod.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    mymethod.div();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    mymethod.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    mymethod.div();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"D").start();
    }
}
```

结果：

![image-20200817000225513](http://picture.youyouluming.cn/image-20200817000225513.png)

可以发现数据突然不在控制内了。造成这个情况的原因又是什么呢？

改进后的代码：

```Java
class Mymethod{
    private int num = 0;
    public synchronized void add() throws InterruptedException {
        while (num != 0) {
            this.wait();
        }
        num++;
        System.out.println(Thread.currentThread().getName() + ":" + num);
        this.notifyAll();
    }

    public synchronized void div() throws InterruptedException {
        while (num == 0) {
            this.wait();
        }
        num--;
        System.out.println(Thread.currentThread().getName() + ":" + num);
        this.notifyAll();
    }
}
public class ConsumerAndProducer {
    public static void main(String[] args) {
        Mymethod mymethod = new Mymethod();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    mymethod.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    mymethod.div();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    mymethod.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    mymethod.div();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        },"D").start();
    }
}
```

改进后的结果

![image-20200817000510432](http://picture.youyouluming.cn/image-20200817000510432.png)

在多线程数据交互判断时，必须使用while做判断，如果使用if做判断，当存在多个消费者和多个生产者时，如果生产者A线程在wait，这时生产者C线程也来到同一个方法中并且进行完了判断，这时A线程被notify唤醒处理完数据，接着C线程也会进来再处理一遍数据，两个生产者处理了数据，而我们期望的是一个生产者对应一个消费者，这时数据就会错乱。使用while判断时，虚假唤醒会进行重新判断是否符合条件，从而避免这一情况。

> 换一种方式解决之前的问题，使用Lock

```Java
class Mymethod2 {
    private int num = 0;
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void add() throws InterruptedException {
        lock.lock();
        try {
            while (num != 0) {
                //this.wait();
                condition.await();
            }
            num++;
            System.out.println(Thread.currentThread().getName() + ":" + num);
            //this.notifyAll();
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void div() throws InterruptedException {
        lock.lock();
        try {
            while (num == 0) {
                //this.wait();
                condition.await();
            }
            num--;
            System.out.println(Thread.currentThread().getName() + ":" + num);
            //this.notifyAll();
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }
}

public class NewConsumerAndProducer {
    public static void main(String[] args) {
        Mymethod2 method = new Mymethod2();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    method.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    method.div();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    method.add();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    method.div();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();
    }
}
```

结果

![image-20200817003404363](http://picture.youyouluming.cn/image-20200817003404363.png)



之前使用synchronized保证线程同步的，现在使用lock上锁unLock解锁来保证线程同步。之前使用wait进入等待并释放资源，使用notifyAll唤醒其他所有等待线程，现在使用Condition的await进入等待，使用signalAll唤醒其他所有线程。

为什么使用Lock？

再次假设一个情况：分别有A、B、C三个线程去调用共享资源，A先调用资源并且打印5次，B再调用资源打印10次，接着C调用打印15次，再回到A....如此循环10次。

代码：

```Java
class ShareResource {
    private int num = 1;
    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();

    public void printFive() {
        lock.lock();
        try {
            if (num != 1) {
                condition1.await();
            }
            for (int i = 1; i <= 5; i++) {
                System.out.println(i);
            }
            num = 2;
            condition2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printTen() {
        lock.lock();
        try {
            if (num != 2) {
                condition2.await();
            }
            for (int i = 1; i <= 10; i++) {
                System.out.println(i);
            }
            num = 3;
            condition3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printFifteen() {
        lock.lock();
        try {
            if (num != 3) {
                condition3.await();
            }
            for (int i = 1; i <= 15; i++) {
                System.out.println(i);
            }
            num = 1;
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}


public class ThreadOrderAccess {
    public static void main(String[] args) {
        ShareResource resource = new ShareResource();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                resource.printFive();
            }
        }).start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                resource.printTen();
            }
        }).start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                resource.printFifteen();
            }
        }).start();
    }
}
```

部分结果：

![image-20200817104136396](http://picture.youyouluming.cn/image-20200817104136396.png)

在这个例子中有个很重要的问题是之前使用synchronized无法做到的，就是精准唤醒，某个方法执行完后固定接着执行下一个方法。而jdk5提供的Lock便可以完成这个工作，使用lock的Condition可以为每个线程分配不同的开关，可以精准唤醒某个线程，这就是使用lock的原因。