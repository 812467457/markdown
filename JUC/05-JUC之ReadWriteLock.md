# JUC之ReadWriteLock（读写锁）

问题引发：之前不管是synchronized或者Lock加锁，都是对资源的所有操作（读和写）加锁，显然这样加锁不合理，因为只有写数据才可能引发数据的错乱，而读操作不会对数据产生影响，但是这样一加锁所有操作都被影响，大大降低执行效率，使用读写锁（ReadWriteLock）来解决这一问题。

没有加锁同时读写的情况：

```Java
class MyResource {
    private volatile Map<String, Object> map = new HashMap<>();

    public void put(String key, Object value) {
        System.out.println(Thread.currentThread().getName() + "写入数据" + key);
        try {
            Thread.sleep(300);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        map.put(key, value);
        System.out.println(Thread.currentThread().getName() + "写入完成");
    }

    public void get(String key) {
        System.out.println(Thread.currentThread().getName() + "读取数据");
        try {
            Thread.sleep(300);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        Object result = map.get(key);
        System.out.println(Thread.currentThread().getName() + "读取完成" + result);
    }
}

public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyResource myResource = new MyResource();
        for (int i = 0; i < 10; i++) {
            final int temp = i;
            new Thread(() -> {
                myResource.put(temp + "", temp + "");
            }).start();
        }

        for (int i = 0; i < 10; i++) {
            final int temp = i;
            new Thread(() -> {
                myResource.get(temp + "");
            }).start();
        }
    }
}
```

部分结果：

![image-20200818004615355](http://picture.youyouluming.cn/image-20200818004615355.png)

出现的问题：严重违背的数据的原子性，写入数据没有完成就接着写入下一条数据了。如果此时加一个lock锁当然可以解决这个问题，但是写和读都被上锁了，执行效率就很低。

使用ReadWriteLock之后

```Java
class MyResource {
    private volatile Map<String, Object> map = new HashMap<>();
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    public void put(String key, Object value) {
        //写操作加上写锁
        readWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "写入数据" + key);
            Thread.sleep(300);
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "写入完成");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            readWriteLock.writeLock().unlock();
        }

    }

    public void get(String key) {
        //为读操作加读锁
        readWriteLock.readLock().lock();

        try {
            System.out.println(Thread.currentThread().getName() + "读取数据");
            Thread.sleep(300);
            Object result = map.get(key);
            System.out.println(Thread.currentThread().getName() + "读取完成" + result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            readWriteLock.readLock().unlock();
        }

    }
}

public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyResource myResource = new MyResource();
        for (int i = 0; i < 10; i++) {
            final int temp = i;
            new Thread(() -> {
                myResource.put(temp + "", temp + "");
            }).start();
        }

        for (int i = 0; i < 10; i++) {
            final int temp = i;
            new Thread(() -> {
                myResource.get(temp + "");
            }).start();
        }
    }
}
```

结果

![image-20200818005813416](http://picture.youyouluming.cn/image-20200818005813416.png)

成功的解决的数据不一致的问题，并且单独给写和读操作分别上锁，保证了程序的运行效率。