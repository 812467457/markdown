# Mybatis入门笔记

## 一、概述
mybatis是一款基于Java的持久层框架。其封装了jdbc的很多细节，使开发者只需要关注sql本身，无需关注注册驱动、建立连接等操作过程。它使用了<u>ORM</u>思想实现了结果集的封装。
ORM：Object Ralational Mappging 对象关系映射。就是把数据库表和实体类及实体类属性对应起来，让我们可以操作实体类就实现操作数据库。
    

## 二、mybatis环境搭建
    1.  创建maven工程，在pom中导入坐标

```xml
<dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.46</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

    2. 创建实体类和dao接口
    3. 创建mybatis的主配置文件 SqlMapConfig.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE configuration        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- mybatis的主配置文件 -->
<configuration>    
    <!-- 配置环境 -->    
    <environments default="development">        
        <!-- 配置MySQL的环境 -->       
        <environment id="development">            
            <!-- 配置事务类型 -->            
            <transactionManager type="JDBC"></transactionManager>            
            <!-- 配置数据源（连接池） -->            
            <dataSource type="POOLED">                
                <!-- 配置连接数据库的基本信息 -->                
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf8"/> 
                <property name="username" value="root"/> 
                <property name="password" value="admin"/> 
            </dataSource>
        </environment>
    </environments>
        <!-- 指定映射配置文件的配置，映射配置文件指的是每个dao的独立配置文件 -->    <mappers>        
    <mapper resource="cn/luming/dao/UserDao.xml"></mapper>   
</mappers>
</configuration>
```

    4. 创建映射配置文件 UserDao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE mapper        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.luming.dao.UserDao">    
    <!-- 配置查询所有  resultType：给出封装到哪里去-->    
    <select id="findAll" resultType="cn.luming.domain.User">       
        select * from user    
    </select>
</mapper>
```

### 环境搭建注意事项
    1. mybatis的映射文件位置必须和dao接口的包结构相同
    2. 映射配置文件mapper标签nameapsce属性值必须是dao接口的全限定类名
    3. 映射配置文件的操作配置，id的属性取值必须是dao接口的方法名
    遵循以上要求，在开发中就无需再写dao的实现类

## 三、入门案例
    1. 读取配置文件
    在日常开发中，绝对路径和相对路径都不常用。经常使用的是类加载器，读取类路径的配置文件或者ServletContext的getRealPath().

```Java
InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
```

    2. 创建sqlSessionFactory工厂,mybatis使用构建者模式创建工厂。构建者模式：把对象的创建细节隐藏，使使用者直接调用方法即可拿到对象。
```Java
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(in);
```

    3. 使用工厂生产一个sqlSession对象。工厂模式：降低了类之间的依赖关系
```Java
SqlSession session = factory.openSession();
```

    4. 使用sqlSession创建dao接口的代理对象。创建dao接口类getMapper实现了动态代理。代理模式：不修改源码的的基础上对已有方法增强
```Java
UserDao userDao = session.getMapper(UserDao.class);
```

    5. 使用代理对象执行方法
```Java
List<User> users = userDao.findAll();
for (User user : users) {    
    System.out.println(user)
;}
```

    6. 释放资源
```Java
session.close();
in.close();
```

* * *

    7. 使用注解配置
    在UserDao的findAll方法上加注解指定语句，把IUserDao.xml移除

```Java
@Select("select * from user")
List<User> findAll();
```
再把SqlMapConfig中指向修改为使用class属性指定被注解的dao全限定类名
```xml
<mappers> 
    <mapper class="cn.luming.dao.UserDao"/>
</mappers>
```

## 四、Mybatis的CRUD
提高代码复用，重复代码整合到注解中
```Java
private InputStream in;
SqlSession sqlSession;
IUserDao userDao;
    @Before//在测试方法执行之前执行
    public void init() throws IOException {    
    //读取配置文件生成字节输入流    
    in = Resources.getResourceAsStream("SqlMapConfig.xml");    
    //获取SqlSessionFactory    
    SqlSessionFactory factory = newSqlSessionFactoryBuilder().build(in);    
    //获取SqlSession对象    
    sqlSession = factory.openSession();    
    //获取dao的代理对象   
    userDao = sqlSession.getMapper(IUserDao.class);
}
@After//在测试方法执行之后执行
public void destroy() throws IOException {    
    //提交事务    
    sqlSession.commit();   
    //释放资源    
    sqlSession.close();   
    in.close();
}
```
### 添加用户方法
    1. 在用户持久层接口加上    
```java
void saveUser(User user);
```

    2. 在映射文件加上

```  Java
<insert id="saveUser"parameterType="cn.luming.domain.User">    
    insert into user(username,address,sex,birthday)values(#{username},#{address},#{sex},#   {birthday});
</insert>
```

    3. 在测试中调用

```Java
@Test
    public void testSaveUser() throws IOException {
        User user = new User();
        user.setUsername("123a");
        user.setAddress("k51k");
        user.setSex("m");
        user.setBirthday(new Date());
        //执行保存方法
        userDao.saveUser(user);
    }
```

     4. 扩展
    获得新增用户id的返回值
    在映射文件的insert标签中插入    
```xml
<!---keyProperty:id的属性名称，对应实体类
        keyColumn：id的列名，对应表
        resultType：结果集类型
        order：什么时候执行改-->
<selectKey keyProperty="id" keyColumn="id" resultType="int" order="AFTER">
            select last_insert_id();
        </selectKey>
```

测试中调用getId()

### 更新用户方法
    1. 在用户持久层接口加上   
```Java
void updateUser(User user);
```

    2. 在映射文件加上
```xml
<update id="updateUser" parameterType="cn.luming.domain.User">
    update user set username=#{username},address=#{address},sex=#{sex},birthday=#{birthday} where id = #{id};
</update>
```

    3. 测试中调用    
```java
 @Test
    public void testUpdateUser() throws Exception {
        User user = new User();
        user.setId(51);
        user.setUsername("awe");
        user.setAddress("nnnn");
        user.setSex("m");
        user.setBirthday(new Date());
        //执行
        userDao.updateUser(user);
    }
```

### 根据id删除用户方法
    1. 在用户持久层接口加上       
```java
void deleteUser(Integer id);
```

    2. 在映射文件加上    
```xml
<delete id="deleteUser" parameterType="java.lang.Integer">    delete from user where id=#{uid}</delete>
```

    3. 测试中调用     
```java
 @Test
    public void testDeleteUser() throws Exception {
        //执行
        userDao.deleteUser(52);
    }
```

### 根据id查询用户方法
    1. 在用户持久层接口加上       
```java
User findById(Integer id);
```

    2. 在映射文件加上    
```xml
<select id="findById" parameterType="java.lang.Integer" resultType="cn.luming.domain.User">
        select * from user where id = #{uid};
    </select>
```

    3. 测试中调用     
```java
@Test
    public void testFindById() throws Exception {
        //执行
        User user = userDao.findById(48);
        System.out.println(user);
    }
```

### 根据名字模糊查询方法
    1. 在用户持久层接口加上       
```java
List<User> findByName(String username);
```

    2. 在映射文件加上    
```xml
<select id="findByName" parameterType="string" resultType="cn.luming.domain.User">
        select * from user where username like #{name};
    </select>
```

    3. 测试中调用     
```java
 @Test
    public void testFindByName() throws Exception {
        //执行
        List<User> userList = userDao.findByName("%王%");
        for (User user : userList) {
            System.out.println(user);
        }
    }
```

    4. 注意：
模糊查询的sql语句也可以写为    
`select * from user where username like '%${value}%'`
测试类中   
`List<User> userList = userDao.findByName("王");`
* 第一种查询最终调用的sql为
`select * from user where username like ?` 
使用的是PrepatedStatement的参数占位符，预处理方式。此种方式使用较多
* 第二种为
`select * from user where username like '%王%'`
使用的是Statement的字符串拼接
  

### 查询总记录人数方法
    1. 在用户持久层接口加上       
```java
int findTotal();
```

    2. 在映射文件加上    
```xml
<select id="findTotal" resultType="java.lang.Integer">
        select count(id) from user ;
    </select>
```

    3. 测试中调用     
```java
 @Test
    public void testFindTotal() throws Exception {
        //执行
        int total = userDao.findTotal();
        System.out.println(total);
    }
```

## 五、结果集映射
为解决数据库字段和实体类属性名称不一致问题
在映射文件中使用resultMap，可以使数据库字段和实体类属性对应起来
```xml
<!-- 结果集映射 -->
    <resultMap id="UserMap" type="User">
        <!--column：数据库中的字段
            property：实体类中的属性
         -->
        <result column="id" property="id"/>
        <result column="username" property="username"/>
        <result column="password" property="password"/>
    </resultMap>
```

## 六、多对一查询
多个学生对应一个老师，查询学生信息并且包括老师
* 子查询
```xml
 <resultMap id="StudentTeacher" type="Student">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
        <!-- 
            属性为对象时用association 
            JavaType用来指定实力类的属性
        -->
        <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
    </resultMap>
    <select id="findAllStudent" resultMap="StudentTeacher">
        select * from student
    </select>
    <select id="getTeacher" resultType="Teacher">
        select * from teacher where id = #{id}
    </select>
```
* 联表查询
```xml
<resultMap id="StudentTeacher2" type="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <association property="teacher" javaType="Teacher">
            <result property="name" column="tname"/>
            <result property="id" column="tid"/>
        </association>
    </resultMap>

    <select id="findAllStudent2" resultMap="StudentTeacher2">
        select s.id sid,s.name sname,t.id tid,t.name tname
        from student s,teacher t
        where s.tid = t.id
    </select>
```
* 查询结果
![de2dc7cebc2186a668f1940833770c5b.png](http://picture.youyouluming.cn/adawdawfawfa.png)

## 七、一对多查询
```xml
<resultMap id="TeacherStudent" type="Teacher">
        <result property="id" column="tid"/>
        <result property="name" column="tname"/>
        <!-- 
            集合用collection
            oftype用来指定映射集合中的实体类型，集合泛型中的类型
        -->
        <collection property="students" ofType="Student">
            <result property="id" column="sid"/>
            <result property="name" column="sname"/>
        </collection>
    </resultMap>
    <select id="findTeacherById" resultMap="TeacherStudent">
        select s.id sid,s.name sname,t.name tname,t.id tid
        from teacher t,student s
        where t.id=s.tid and tid=#{tid}
    </select>
```

## 八、动态SQL
### 1、if
通过给定的条件查询结果，给定的条件是不一定，使用if判断给出的什么条件，再执行对应的SQL
```xml
<!-- 根据条件查询bolg -->
<select id="findBlogByCondition" resultType="Blog" parameterType="map">
        select * from blog
        <where>
            <if test="title != null">
                and title = #{title}
            </if>
            <if test="author != null">
                and author = #{author}
            </if>
        </where>
    </select>
```

### 2、choose (when, otherwise)
有时候，我们不想使用所有的条件，而只是想从多个条件中选择一个使用。针对这种情况，MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。
```xml
<select id="findBlogByCondition2" resultType="Blog" parameterType="map">
        select * from blog
       <where>
           <choose>
               <when test="title != null">
                   and title = #{title}
               </when>
               <when test="author != null">
                   and author = #{author}
               </when>
               <otherwise>
                   and views = #{views}
               </otherwise>
           </choose>
       </where>
    </select>
```

### 3、trim (where, set)
1. where
如果没有where，SQL将会变成select * from blog and title = #{title}，此SQL为错误的写法。使用
where 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，where 元素也会将它们去除。

2. set
set 元素会动态地在行首插入 SET 关键字，并会删掉额外的逗号（这些逗号是在使用条件语句给列赋值时引入的）。
```xml
<update id="updateBlog" parameterType="map">
        update blog
        <set>
        <!-- 此时如果只有title属性赋值，set标签会自动删除title后面多余的逗号 -->
            <if test="title != null">
                title = #{title},
            </if>
            <if test="author != null">
                author = #{author}
            </if>
            where id = #{id}
        </set>
    </update>
```

3. trim
使用trim标签自定义set和where元素的功能
```xml
<select id="findBlogByCondition" resultType="Blog" parameterType="map">
    select * from blog
    <!-- 和上面的WHERE功能一样 -->
    <trim prefix="WHERE" prefixOverrides="AND|OR">
        <if test="title != null">
            and title = #{title}
        </if>
        <if test="author != null">
            and author = #{author}
        </if>
    </trim>
```

```xml
<update id="updateBlog" parameterType="map">
        update blog
        <!-- 和上面set的功能一样 -->
        <trim prefix="SET" suffixOverrides=",">
            <if test="title != null">
                title = #{title},
            </if>
            <if test="author != null">
                author = #{author}
            </if>
            where id = #{id}
        </trim>
    </update>
```



### 4、foreach
动态 SQL 的另一个常见使用场景是对集合进行遍历（尤其是在构建 IN 条件语句的时候）
```xml
<!-- 
    collection：要遍历的集合
    item：本次迭代获取到的元素
    open：开头的字符串
    close：结尾的字符串
    separator：迭代之间的分隔符
-->
<select id="findUserForeach" resultType="User" parameterType="map">
        select * from user
        <where>
            <foreach collection="ids" item="id" open="(" close=")" separator="or">
                id = #{id}
            </foreach>
        </where>
    </select>
```

### 5、SQL片段
使用sql标签抽取公共sql语句，使用include标签引入公共的sql语句
```xml
<sql id="if-title-author">
        <if test="title != null">
            and title = #{title}
        </if>
        <if test="author != null">
            and author = #{author}
        </if>
    </sql>
    <select id="findBlogByCondition" resultType="Blog" parameterType="map">
        select * from blog
        <where>
            <include refid="if-title-author"></include>
        </where>
    </select>
```

## 九、mybatis缓存
### 1、一级缓存
默认情况下是一级缓存，在一个sqlsession中反复使用查询语句，sql只会执行一次，使用的是缓存中的数据。但是使用增删改操作，会清除sqlseeion中的一级缓存。
### 2、二级缓存
要启用全局的二级缓存，只需要在你的 SQL 映射文件中添加一行：
```xml
<cache/>
```
效果如下
* 映射语句文件中的所有 select 语句的结果将会被缓存。
* 映射语句文件中的所有 insert、update 和 delete 语句会刷新缓存。
* 缓存会使用最近最少使用算法（LRU, Least Recently Used）算法来清除不需要的缓存。
* 缓存不会定时进行刷新（也就是说，没有刷新间隔）。
* 缓存会保存列表或对象（无论查询方法返回哪种）的 1024 个引用。
* 缓存会被视为读/写缓存，这意味着获取到的对象并不是共享的，可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。
这些属性可以通过 cache 元素的属性来修改。
```xml
<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```
这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512 个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突。
可用的清除策略有：
* LRU – 最近最少使用：移除最长时间不被使用的对象。
* FIFO – 先进先出：按对象进入缓存的顺序来移除它们。
* SOFT – 软引用：基于垃圾回收器状态和软引用规则移除对象。
* WEAK – 弱引用：更积极地基于垃圾收集器状态和弱引用规则移除对象。

默认的清除策略是 LRU。
flushInterval（刷新间隔）属性可以被设置为任意的正整数，设置的值应该是一个以毫秒为单位的合理时间量。 默认情况是不设置，也就是没有刷新间隔，缓存仅仅会在调用语句时刷新。
size（引用数目）属性可以被设置为任意正整数，要注意欲缓存对象的大小和运行环境中可用的内存资源。默认值是 1024。
readOnly（只读）属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓存对象的相同实例。 因此这些对象不能被修改。这就提供了可观的性能提升。而可读写的缓存会（通过序列化）返回缓存对象的拷贝。 速度上会慢一些，但是更安全，因此默认值是 false。
提示 :二级缓存是事务性的。这意味着，当 SqlSession 完成并提交时，或是完成并回滚，但没有执行 flushCache=true 的 insert/delete/update 语句时，缓存会获得更新。

### 3、自定义缓存
除了上述自定义缓存的方式，你也可以通过实现你自己的缓存，或为其他第三方缓存方案创建适配器，来完全覆盖缓存行为。


## 十、配置优化
### 1、属性优化
引入外部的配置文件
```xml
 <!-- 引入配置文件 -->
 <properties resource="jdbc.properties"/>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>
```
### 2、别名优化
在主配置文件中配置别名，映射文件中就可以使用改别名，避免了全限定类名的麻烦
```xml
       <typeAliases>
       <!-- 在映射文件中直接使用user即可 -->    
        <typeAlias type="cn.luming.domain.User" alias="User"/>
        <!-- 指定一个包名，在没有注解的情况下，会默认使用JavaBean的首字母小写的非限定类名作为它的别名，如果有注解，则别名为注解上的 -->
        <package name="cn.luming.domain"/>
    </typeAliases>
```



## 十一、mybatis的日志工厂
![0aaf9169760aa142757e953795cf7411.png](http://picture.youyouluming.cn/afawfawe214afa.png)
### 1、标准日志
在主配置文件中配置日志
```xml
 <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
```
之后运行就会显示日志信息
![502ef388cef5f0f33acedc508eda60bd.png](http://picture.youyouluming.cn/awfaw1312qagw3.png)

### 2、log4j
1. 导log4j的包
```xml
<dependencies>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
    </dependencies>
```
2. 配置log4j.properties文件
```properties
#将等级为DEBUG的日志信息输出到控制台和file两个目的地
log4j.rootLogger=DEBUG,console,file
#控制台输出设置
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.Target=System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern= [%c]-%m%n
#文件输出设置
log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/luming.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n
#日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```
3. 在核心配置文件配置log4j为日志实现
```xml
<settings>    
    <setting name="logImpl" value="LOG4J"/>
</settings>
```

## 十二、mybatis连接池
 mybatis连接池提供的三种连接方式
* 配置的位置：
    主配置文件SqlMapconfig.xml中的dataSource标签，type属性就采用哪种连接池方式
* type属性的取值：
    1. POOLED：采用传统的javax.sql.DataSourse规范的连接池，mybatis中针对规范的实现。从连接池中获取连接，使用完归还连接池。
    2. UNPOOLED：采用传统获取连接的方式，虽然也实现了javax.sql.DataSourse接口，但是没有使用池的思想。没有容器的概念，每一次使用都会创建新的连接。
    3. JNDI：采用服务器提供的JNDI技术实现，来获取DataSourse对象，不同服务器所获得的DataSourse不同。注意：只有web或maven的war工程可以使用。tomcat服务器使用的是dbcp连接池。
    
    
## 十三、mybatis事务 
* mybatis的事务是通过sqlSession对象的conmit方法和rollback方法实现事务的提交和回滚
* 通过openSession(true)方法传入true值即可自动提交 事务



​    











​    



​    