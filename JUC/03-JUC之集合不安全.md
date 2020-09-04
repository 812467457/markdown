# JUC之集合不安全

## 一、ArrayList线程不安全问题

看一段代码

```Java
public class NotSafe01 {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();

        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0, 8));
                System.out.println(list);
            }).start();
        }
    }
}
```

执行结果

![image-20200817144712572](http://picture.youyouluming.cn/image-20200817144712572.png)

因为ArrayList是线程不安全的，当多个线程同时对ArrayList进行读写操作时就会发生ConcurrentModificationException（并发修改）异常



### 1、解决方法一：改用线程安全的Vector

ArrayList的源码

![image-20200817144952934](http://picture.youyouluming.cn/image-20200817144952934.png)

Vector的源码

![image-20200817145036255](http://picture.youyouluming.cn/image-20200817145036255.png)

改用Vector虽然保证了线程安全，但是Vector的性能没有ArrayList高，所以此方法不是首选

### 2、解决方法二：使用Collections类的synchronizedList方法

代码如下

```Java
public class NotSafe01 {
    public static void main(String[] args) {
        List<String> list = Collections.synchronizedList(new ArrayList<>());

        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0, 8));
                System.out.println(list);
            }).start();
        }
    }
}
```

结果

![image-20200817152412740](http://picture.youyouluming.cn/image-20200817152412740.png)

Collections可以把不安全的线程转换为安全的，因为使用synchronized保证线程同步，所以也会影响效率。

### 3、解决方法三：使用JUC中的CopyOnWriteArrayList类

代码：

```Java
public class NotSafe01 {
    public static void main(String[] args) {
        List<String> list = new CopyOnWriteArrayList<>();

        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                list.add(UUID.randomUUID().toString().substring(0, 8));
                System.out.println(list);
            }).start();
        }
    }
}
```

写时复制：读写分离思想的一种体现，读和写不是同一个容器，保证写数据只能一个线程写，读数据可以多个线程读。

CopyOnWriteArrayList的add方法源码分析

![image-20200817162356642](http://picture.youyouluming.cn/image-20200817162356642.png)

添加元素时先加锁，保证只能一个线程操作add，然后创建一个数组，把原来集合中的元素复制到这个数组中，再对其进行写操作，这样不会影响其他线程的读操作，既能保证线程安全，又能保证高效，最后再把新的数组设置回去，接着放开锁。

## 二、HashSet线程不安全问题

多线程操作hashSet集合时一样也存在安全问题

代码

```Java
public class NotSafe02 {
    public static void main(String[] args) {
        Set<String> set = new HashSet<>();

        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                set.add(UUID.randomUUID().toString().substring(0, 8));
                System.out.println(set);
            }).start();
        }
    }
}
```

结果

![image-20200817163624427](http://picture.youyouluming.cn/image-20200817163624427.png)



### 1、解决方式一：使用Collections类的synchronizedList方法

代码：

```Java
public class NotSafe02 {
    public static void main(String[] args) {
        Set<String> set = Collections.synchronizedSet(new HashSet<>());

        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                set.add(UUID.randomUUID().toString().substring(0, 8));
                System.out.println(set);
            }).start();
        }
    }
}
```

### 2、解决方式二：使用JUC的CopyOnWriteArraySet

代码：

```Java
public class NotSafe02 {
    public static void main(String[] args) {
        Set<String> set = new CopyOnWriteArraySet<>();

        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                set.add(UUID.randomUUID().toString().substring(0, 8));
                System.out.println(set);
            }).start();
        }
    }
}
```

原理同ArrayList解决方式三一样。



### 3、HashSet复习

问：HashSet底层数据结构是什么？

答：HashSet底层数据结构是hashMap。

问：底层是map按道理说添加元素时应该添加的键值对，但HashSet却添加的一个值，什么原因？

答：因为HashSet的add方法就是调用的hashmap的put方法，传进去真正的数据旧是key，value是一个写死的Object类型的常量。



## 三、HashMap线程不安全问题

和其他集合相同的安全问题

代码：

```Java
public class NotSafe03 {
    public static void main(String[] args) {
        Map<String,String> map = new HashMap<>();

        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                map.put(Thread.currentThread().getName(),UUID.randomUUID().toString().substring(0, 8));
                System.out.println(map);
            }).start();
        }
    }
}
```

结果：

![image-20200817172907246](http://picture.youyouluming.cn/image-20200817172907246.png)

### 1、解决方式一：使用Collections类的synchronizedMap方法

```java
public class NotSafe03 {
    public static void main(String[] args) {
        Map<String,String> map = Collections.synchronizedMap(new HashMap<>());

        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                map.put(Thread.currentThread().getName(),UUID.randomUUID().toString().substring(0, 8));
                System.out.println(map);
            }).start();
        }
    }
}
```

### 2、解决方式二：使用JUC的ConcurrentHashMap

```java 
public class NotSafe03 {
    public static void main(String[] args) {
        Map<String,String> map = new ConcurrentHashMap<>();

        for (int i = 0; i < 30; i++) {
            new Thread(() -> {
                map.put(Thread.currentThread().getName(),UUID.randomUUID().toString().substring(0, 8));
                System.out.println(map);
            }).start();
        }
    }
}
```



