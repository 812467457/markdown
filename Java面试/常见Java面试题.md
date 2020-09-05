# JavaSE

## 基础部分

### String、StringBuffer、StringBuilder的区别

String是使用final修饰的，所以在定义后不可以修改，每次修改都是重新创建一个String对象。

StringBuffer是一个可变的字符串，并且线程安全，但是效率过低。

StringBuilder和StringBuffer相同，但是不是线程安全的，相对效率较高。

如果字符串的大小确定，尽量在创建字符串的同时指定大小。



### final关键字怎么用的

final修饰一个值时，这个值不可以变。

final修饰一个引用对象时，这个引用地址不可以变。

final修饰一个方法时，这个方法不可以被继承修改。

final修饰一个类时，这个类不能被继承。并且final类中的成员变量默认也是final修饰的



## 集合

### HashMap和Hashtable的区别

1. HashMap继承于AbstractMap，Hashtable继承于Dictionary，两者都实现了Map接口
2. HashMap运行Key和Value为null，Hashtable不可以
3. HashMap是线程不安全的，Hashtable是线程安全的



### 谈谈HashMap的put方法

HashMap的初始值是16，加载因子是0.75，并且要求该容量必须是2的整数次幂。

在1.7中HashMap的数据结构是数组+链表，扩容采用的插头法，可能造成闭环，线程不安全。

在1.8中HashMap的数据结构是数组+链表+红黑树，在链表的长度大于8时，回自动转为红黑树（如果当前容量小于64，会先扩容）。当红黑树元素少于6个时，会再次从红黑树转为链表。在多线程put操作下，会出现数据被覆盖的情况，同样的线程不安全。



### ArrayList和LinkList的区别

ArrayList底层数据结构时数组，LinkList底层数据结构是双向链表。

因为ArrayList是数组，所以在获取元素时直接返回数组下标对应的元素即可，但是数组的空间都是连续的，如果修改数据就会牵扯到位置的挪动，因此ArrayList的修改效率低。

LinkList的底层数据结构是双向链表，链表没有下标，所以查询的时候会从第一个元素开始查询，效率过低，但是链表在修改元素时只需要移动链表对应的指针就可以，效率相对高一些。



# JavaWeb



# Spring

### Spring Bean生命周期





# Mybatis



# SpringMVC



# SpringBoot

