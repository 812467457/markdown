# Stream流

## 一、Stream流是什么

是数据渠道用于操作数据源（集合数组等）所生成的元素序列

**集合讲的的是数据，流讲的是计算**



## 二、Stream流的特点

* Stream不会自己存储元素
* Stream不会改变源对象。相反，他们会返回一个持有结果的新stream
* stream操作时延迟执行的，表示等到需要结果的时候才执行



## 三、使用Stream

### 1、操作步骤

1. 创建一个stream：一个数据源（数组、集合）
2. 中间操作：处理数据源数据
3. 终止操作：执行中间操作链，产生结果



### 2、代码

```Java
@Data
@AllArgsConstructor
@NoArgsConstructor
@ToString
class User {
    private int id;
    private String name;
    private int age;
}

public class StreamDemo {
    public static void main(String[] args) {
        User user1 = new User(1, "a", 23);
        User user2 = new User(2, "b", 24);
        User user3 = new User(3, "c", 22);
        User user4 = new User(4, "d", 28);
        User user5 = new User(6, "e", 26);

        List<User> list = Arrays.asList(user1, user2, user3, user4, user5);

        //创建stream,操作数据
        list.stream().filter(user -> {
            //筛选userId为偶数的
            return user.getId() % 2 == 0;
        }).filter(user -> {
            //筛选userAge>24的
            return user.getAge() > 24;
        }).map((user) -> {
            //把名字转大写
            return user.getName().toUpperCase();
        }).sorted((o1, o2) -> {
            //对名字进行倒序排序
            return o2.compareTo(o1);
            //limit(1)最后只输出一个
        }).limit(1).forEach(System.out::println);
    }
}
```

