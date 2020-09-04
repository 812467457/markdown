# ssm整合
## 一、搭建整合环境  （maven）
### 1、 搭建整合环境  
1. 整合说明：SSM整合可以使用多种方式，选择XML + 注解的方式
2. 整合的思路
    1. 先搭建整合的环境
    2. 先把Spring的配置搭建完成
    3. 再使用Spring整合SpringMVC框架
    4. 最后使用Spring整合MyBatis框架
3. 创建数据库和表结构
    语句
    ```sql
     create database ssm;
    use ssm;
    create table account(
        id int primary key auto_increment,
        name varchar(20),
        money double
    );
    ```


4. 创建maven的web工程
    在pom文件中导入坐标
    ```xml
      <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <spring.version>5.0.2.RELEASE</spring.version>
        <slf4j.version>1.6.6</slf4j.version>
        <log4j.version>1.2.12</log4j.version>
        <mysql.version>5.1.6</mysql.version>
        <mybatis.version>3.4.5</mybatis.version>
      </properties>

      <dependencies>
        <dependency>
          <groupId>org.aspectj</groupId>
          <artifactId>aspectjweaver</artifactId>
          <version>1.6.8</version>
        </dependency>
        <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-aop</artifactId>
          <version>${spring.version}</version>
        </dependency>
        <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
          <version>${spring.version}</version>
        </dependency>
        <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-web</artifactId>
          <version>${spring.version}</version>
        </dependency>
        <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-webmvc</artifactId>
          <version>${spring.version}</version>
        </dependency>
        <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-test</artifactId>
          <version>${spring.version}</version>
        </dependency>
        <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-tx</artifactId>
          <version>${spring.version}</version>
        </dependency>
        <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-jdbc</artifactId>
          <version>${spring.version}</version>
        </dependency>
        <dependency>
          <groupId>junit</groupId>
          <artifactId>junit</artifactId>
          <version>4.12</version>
          <scope>compile</scope>
        </dependency>
        <dependency>
          <groupId>mysql</groupId>
          <artifactId>mysql-connector-java</artifactId>
          <version>${mysql.version}</version>
        </dependency>
        <dependency>
          <groupId>javax.servlet</groupId>
          <artifactId>servlet-api</artifactId>
          <version>2.5</version>
          <scope>provided</scope>
        </dependency>
        <dependency>
          <groupId>javax.servlet.jsp</groupId>
          <artifactId>jsp-api</artifactId>
          <version>2.0</version>
          <scope>provided</scope>
        </dependency>
        <dependency>
          <groupId>jstl</groupId>
          <artifactId>jstl</artifactId>
          <version>1.2</version>
        </dependency>
        <!-- log4j start -->
        <dependency>
          <groupId>log4j</groupId>
          <artifactId>log4j</artifactId>
          <version>${log4j.version}</version>
        </dependency>
        <dependency>
          <groupId>org.slf4j</groupId>
          <artifactId>slf4j-api</artifactId>
          <version>${slf4j.version}</version>
        </dependency>
        <dependency>
          <groupId>org.slf4j</groupId>
          <artifactId>slf4j-log4j12</artifactId>
          <version>${slf4j.version}</version>
        </dependency>
        <!-- log end -->
        <dependency>
          <groupId>org.mybatis</groupId>
          <artifactId>mybatis</artifactId>
          <version>${mybatis.version}</version>
        </dependency>
        <!-- spring整合mybatis -->
        <dependency>
          <groupId>org.mybatis</groupId>
          <artifactId>mybatis-spring</artifactId>
          <version>1.3.0</version>
        </dependency>
        <dependency>
          <groupId>com.alibaba</groupId>
          <artifactId>druid</artifactId>
          <version>1.1.20</version>
        </dependency>
      </dependencies>

      <build>
        <finalName>ssm02</finalName>
        <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
          <plugins>
            <plugin>
              <artifactId>maven-clean-plugin</artifactId>
              <version>3.1.0</version>
            </plugin>
            <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->
            <plugin>
              <artifactId>maven-resources-plugin</artifactId>
              <version>3.0.2</version>
            </plugin>
            <plugin>
              <artifactId>maven-compiler-plugin</artifactId>
              <version>3.8.0</version>
            </plugin>
            <plugin>
              <artifactId>maven-surefire-plugin</artifactId>
              <version>2.22.1</version>
            </plugin>
            <plugin>
              <artifactId>maven-war-plugin</artifactId>
              <version>3.2.2</version>
            </plugin>
            <plugin>
              <artifactId>maven-install-plugin</artifactId>
              <version>2.5.2</version>
            </plugin>
            <plugin>
              <artifactId>maven-deploy-plugin</artifactId>
              <version>2.8.2</version>
            </plugin>
          </plugins>
        </pluginManagement>
      </build>
    </project>
    ```

5. 编写实体类
    ```java
    public class Account implements Serializable {
        private Integer id;
        private String name;
        private Double money;
        public Integer getId() {
            return id;
        }
        public void setId(Integer id) {
            this.id = id;
        }
        public String getName() {
            return name;
        }
        public void setName(String name) {
            this.name = name;
        }
        public Double getMoney() {
            return money;
        }
        public void setMoney(Double money) {
            this.money = money;
        }
        @Override
        public String toString() {
            return "Account{" +
                    "id=" + id +
                    ", name='" + name + '\'' +
                    ", money=" + money +
                    '}';
        }
    }
    ```

6. 编写dao接口
    ```java
    public interface IAccountDao {
        List<Account> findAll();
        void saveAccount (Account account);
    }
    ```

7. 编写service接口和实现类
    ```java
    //接口
    public interface IAccountService {
        List<Account> findAll();
        void saveAccount(Account account);
    }

    //实现类
    @Service(value = "accountSerivce")
    public class AccountServiceImpl implements IAccountService {
        @Override
        public List<Account> findAll() {
            System.out.println("service findAll");
            return null;
        }
        @Override
        public void saveAccount(Account account) {
            System.out.println("service saveAccount");
        }
    }
    ```
    
8. 编写controller类
    ```java
    @Controller
    public class AccountController {
    }
    ```

## 二、Spring框架代码编写
### 1、搭建和测试Spring的开发环境

1. 创建applicationContext.xml.xml文件，并编写配置信息，**在Service类上加注解**

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:aop="http://www.springframework.org/schema/aop"
           xmlns:tx="http://www.springframework.org/schema/tx"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop.xsd
           http://www.springframework.org/schema/tx
           http://www.springframework.org/schema/tx/spring-tx.xsd">
        <!-- 开启注解扫描  -->
        <context:component-scan base-package="cn.luming">
            <!-- 配置不需要扫描的注解 -->
            <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        </context:component-scan>
    </beans>
    ```
    
2. 编写测试方法
    ```java
    public class SpringTest {
        @Test
        public void test01(){
            ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
            IAccountService accountService = ac.getBean("accountService", IAccountService.class);
            accountService.findAll();
        }
    }
    ```

## 三、Spring整合SpringMVC框架
### 1、搭建和测试SpringMVC的开发环境

1. 在web.xml中配置DispatcherServlet前端控制器

    ```xml
    <!DOCTYPE web-app PUBLIC
            "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
            "http://java.sun.com/dtd/web-app_2_3.dtd" >

    <web-app>
        <display-name>Archetype Created Web Application</display-name>
        <!-- 解决中文乱码的过滤器 -->
        <filter>
            <filter-name>encodingFilter</filter-name>
            <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
            <!-- 自定义编码集 -->
            <init-param>
                <param-name>encoding</param-name>
                <param-value>UTF-8</param-value>
            </init-param>
        </filter>
        <filter-mapping>
            <filter-name>encodingFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>
        <!-- 配置前端控制器 -->
        <servlet>
            <servlet-name>dispatcherServlet</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
            <!-- 加载springMVC配置文件 -->
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>classpath:springmvc.xml</param-value>
            </init-param>
            <!-- 启动服务器创建servlet -->
            <load-on-startup>1</load-on-startup>
        </servlet>
        <servlet-mapping>
            <servlet-name>dispatcherServlet</servlet-name>
            <url-pattern>/</url-pattern>
        </servlet-mapping>
    </web-app>
    ```
2. 创建springmvc.xml配置文件,并配置

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:mvc="http://www.springframework.org/schema/mvc"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:aop="http://www.springframework.org/schema/aop"
           xmlns:tx="http://www.springframework.org/schema/tx"
           xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/aop
            http://www.springframework.org/schema/aop/spring-aop.xsd
            http://www.springframework.org/schema/tx
            http://www.springframework.org/schema/tx/spring-tx.xsd">
        <!-- 开启注解扫描 -->
        <context:component-scan base-package="cn.luming">
            <!-- 只扫描Controller -->
            <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        </context:component-scan>

        <!-- 配置视图解析器 -->
        <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <!-- 文件路径 -->
            <property name="prefix" value="/WEB-INF/pages/"/>
            <!-- 文件后缀 -->
            <property name="suffix" value=".jsp"/>
        </bean>

        <!-- 过滤静态资源 -->
        <mvc:resources mapping="/js/**" location="/js/"/>

        <!-- 开启mvc注解支持 -->
        <mvc:annotation-driven/>
    </beans>
    ```

3. 测试springmvc环境

    1. 编写index.jsp和要跳转的success.jsp

        ```html
        <a href="account/findAll">findAll</a>
        ```
    2. 编写Controller类，记得配置tomcat

        ```java
        @Controller
        @RequestMapping(value = "account")
        public class AccountController {
            @RequestMapping(value = "findAll")
            public ModelAndView findAll(){
                System.out.println("Controller findAll");
                ModelAndView mv = new ModelAndView();
                mv.setViewName("success");
                return mv;
            }
        }
        ```

### 2、Spring整合SpringMVC的框架

1. 目的：在controller中能成功的调用service对象中的方法。
2. 在项目启动的时候，就去加载applicationContext.xml的配置文件，在web.xml中配置
ContextLoaderListener监听器（该监听器只能加载WEB-INF目录下的applicationContext.xml的配置文
件）。
    ```xml
    <!-- 配置spring监听器 -->
        <listener>
            <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>
        <!-- 配置加载文件的类路径 -->
        <context-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </context-param>
    ```
3. 在controller中注入service对象，调用service对象的方法进行测试
    ```java
    @Autowired
        private IAccountService accountService;
        @RequestMapping(value = "/findAll")
        public ModelAndView findAll(){
            System.out.println("Controller findAll");
            accountService.findAll();
            ModelAndView mv = new ModelAndView();
            mv.setViewName("success");
            return mv;
        }
    ```

## 四、Spring整合MyBatis框架
### 1、搭建和测试MyBatis的环境
1. 在web项目中编写SqlMapConfig.xml的配置文件，编写核心配置文件
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <!-- 配置环境 -->
        <environments default="mysql">
            <environment id="mysql">
                <transactionManager type="JDBC"/>
                <dataSource type="POOLED">
                    <property name="driver" value="com.mysql.jdbc.Driver"/>
                    <property name="url" value="jdbc:mysql://localhost:3306/ssm?characterEncoding=utf8"/>
                    <property name="username" value="root"/>
                    <property name="password" value="admin"/>
                </dataSource>
            </environment>
        </environments>
        <!-- 引入配置文件,使用注解 -->
        <mappers>
            <package name="cn.luming.dao"/>
        </mappers>
    </configuration>
    ```

2. 在AccountDao接口的方法上添加注解，编写SQL语句
    ```java
    public interface IAccountDao {
        @Select("select * from account")
        List<Account> findAll();
        @Insert("insert into account (name,money) values (#{name},#{money})")
        void saveAccount (Account account);
    }
    ```
    
3. 编写测试的方法
    ```java
    //查询所有
    @Test
        public void test01() throws IOException {
            //加载配置文件
            InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
            //创建工厂
            SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(in);
            //创建session对象
            SqlSession sqlSession = sessionFactory.openSession();
            //获取代理对象
            IAccountDao mapper = sqlSession.getMapper(IAccountDao.class);
            //执行方法
            List<Account> accounts = mapper.findAll();
            for (Account account : accounts) {
                System.out.println(account);
            }
            sqlSession.close();
            in.close();
        }
        
        //保存
        @Test
        public void test02() throws IOException {
            //加载配置文件
            InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
            //创建工厂
            SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(in);
            //创建session对象
            SqlSession sqlSession = sessionFactory.openSession();
            //获取代理对象
            IAccountDao mapper = sqlSession.getMapper(IAccountDao.class);
            Account account = new Account();
            account.setName("qqq");
            account.setMoney(5000D);
            mapper.saveAccount(account);
            //提交事务
            sqlSession.commit();
            sqlSession.close();
            in.close();
    }
    ```

### 2、Spring整合MyBatis框架
1. 把SqlMapConfig.xml配置文件中的内容配置到applicationContext.xml配置文件中
    ```xml
    <!-- spring整合mybatis -->
        <!-- 配置连接池 -->
        <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
            <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
            <property name="url" value="jdbc:mysql://localhost:3306/ssm?characterEncoding=utf8"/>
            <property name="username" value="root"/>
            <property name="password" value="admin"/>
        </bean>
        <!-- 配置slqSession工厂 -->
        <bean id="sessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
            <property name="dataSource" ref="dataSource"/>
        </bean>
        <!-- 配置接口所在包 -->
        <bean id="mapperScanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
            <property name="basePackage" value="cn.luming.dao"/>
        </bean>
    ```
2. 在AccountDao接口中添加@Repository注解
3. 在service中注入dao对象，进行测试

    ```java
    @Service(value = "accountService")
    public class AccountServiceImpl implements IAccountService {
        @Autowired
        IAccountDao accountDao;

        @Override
        public List<Account> findAll() {
            accountDao.findAll();
            System.out.println("service findAll");
            return null;
        }

        @Override
        public void saveAccount(Account account) {
            accountDao.saveAccount(account);
            System.out.println("service saveAccount");
        }
    }
    ```

    ```java
    @Controller
    @RequestMapping(value = "/account")
    public class AccountController {
        @Autowired
        private IAccountService accountService;
        @RequestMapping(value = "/findAll")
        public ModelAndView findAll() {
            System.out.println("Controller findAll");
            List<Account> accounts = accountService.findAll();
            ModelAndView mv = new ModelAndView();
            for (Account account : accounts) {
                System.out.println(account);
            }
            mv.addObject("accounts", accounts);
            mv.setViewName("success");
            return mv;
        }
    }
    ```
    jsp页面展示数据

    ```html
    <c:forEach items="${accounts}" var="account">
        ${account.name}
        ${account.money}
    </c:forEach>
    ```

### 3、配置Spring的声明式事务管理
1. 在applicationContext.xml中配置

    ```xml
    <!-- 配置spring声明式事务管理 -->
        <!-- 配置事务管理器 -->
        <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource"/>
        </bean>
        <!-- 配置事务通知 -->
        <tx:advice id="txAdvice" transaction-manager="transactionManager">
            <tx:attributes>
                <tx:method name="find*" read-only="true"/>
                <tx:method name="*" isolation="DEFAULT"/>
            </tx:attributes>
        </tx:advice>
        <!-- 配置AOP增强 -->
        <aop:config>
            <aop:advisor advice-ref="txAdvice" pointcut="execution(public * cn.luming.service.impl.*.*(..))"/>
        </aop:config>
    ```

2. 调用测试

    ```java
     @RequestMapping(value = "/saveAccount")
        public ModelAndView saveAccount(Account account) {
            System.out.println("Controller saveAccount");
            accountService.saveAccount(account);
            ModelAndView mv = new ModelAndView();
            mv.setViewName("list");
            return mv;
        }
    ```
    jsp表单页面
    ```html
    <form action="account/saveAccount" method="post">
        姓名:<input type="text" name="name"><br>
        金额:<input type="text" name="money"><br>
        <input type="submit" value="提交">
    </form>
    ```
    
## 五、所需的配置文件
1. web.xml
    ```xml
    <!DOCTYPE web-app PUBLIC
            "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
            "http://java.sun.com/dtd/web-app_2_3.dtd" >

    <web-app>
        <display-name>Archetype Created Web Application</display-name>
        <!-- 配置加载文件的类路径 -->
        <context-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:applicationContext.xml</param-value>
        </context-param>
        <!-- 解决中文乱码的过滤器 -->
        <filter>
            <filter-name>encodingFilter</filter-name>
            <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
            <!-- 自定义编码集 -->
            <init-param>
                <param-name>encoding</param-name>
                <param-value>UTF-8</param-value>
            </init-param>
        </filter>
        <filter-mapping>
            <filter-name>encodingFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>
        <!-- 配置spring监听器 -->
        <listener>
            <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>
        <!-- 配置前端控制器 -->
        <servlet>
            <servlet-name>dispatcherServlet</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
            <!-- 加载springMVC配置文件 -->
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>classpath:springmvc.xml</param-value>
            </init-param>
            <!-- 启动服务器创建servlet -->
            <load-on-startup>1</load-on-startup>
        </servlet>
        <servlet-mapping>
            <servlet-name>dispatcherServlet</servlet-name>
            <url-pattern>/</url-pattern>
        </servlet-mapping>

    </web-app>
    ```

2. applicationContext.xml
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns:context="http://www.springframework.org/schema/context"
               xmlns:aop="http://www.springframework.org/schema/aop"
               xmlns:tx="http://www.springframework.org/schema/tx"
               xsi:schemaLocation="http://www.springframework.org/schema/beans
               http://www.springframework.org/schema/beans/spring-beans.xsd
               http://www.springframework.org/schema/context
               http://www.springframework.org/schema/context/spring-context.xsd
               http://www.springframework.org/schema/aop
               http://www.springframework.org/schema/aop/spring-aop.xsd
               http://www.springframework.org/schema/tx
               http://www.springframework.org/schema/tx/spring-tx.xsd">
            <!-- 开启注解扫描  -->
            <context:component-scan base-package="cn.luming">
                <!-- 配置不需要扫描的注解 -->
                <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
            </context:component-scan>

            <!-- spring整合mybatis -->
            <!-- 配置连接池 -->
            <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
                <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/ssm?characterEncoding=utf8"/>
                <property name="username" value="root"/>
                <property name="password" value="admin"/>
            </bean>
            <!-- 配置slqSession工厂 -->
            <bean id="sessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
                <property name="dataSource" ref="dataSource"/>
            </bean>
            <!-- 配置接口所在包 -->
            <bean id="mapperScanner" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
                <property name="basePackage" value="cn.luming.dao"/>
            </bean>
            <!-- 配置spring声明式事务管理 -->
            <!-- 配置事务管理器 -->
            <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
                <property name="dataSource" ref="dataSource"/>
            </bean>
            <!-- 配置事务通知 -->
            <tx:advice id="txAdvice" transaction-manager="transactionManager">
                <tx:attributes>
                    <tx:method name="find*" read-only="true"/>
                    <tx:method name="*" isolation="DEFAULT"/>
                </tx:attributes>
            </tx:advice>
            <!-- 配置AOP增强 -->
            <aop:config>
                <aop:advisor advice-ref="txAdvice" pointcut="execution(public * cn.luming.service.impl.*.*(..))"/>
            </aop:config>
        </beans>
    ```
3. springmvc.xml
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:mvc="http://www.springframework.org/schema/mvc"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xmlns:aop="http://www.springframework.org/schema/aop"
           xmlns:tx="http://www.springframework.org/schema/tx"
           xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/mvc
            http://www.springframework.org/schema/mvc/spring-mvc.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd
            http://www.springframework.org/schema/aop
            http://www.springframework.org/schema/aop/spring-aop.xsd
            http://www.springframework.org/schema/tx
            http://www.springframework.org/schema/tx/spring-tx.xsd">
        <!-- 开启注解扫描 -->
        <context:component-scan base-package="cn.luming">
            <!-- 只扫描Controller -->
            <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        </context:component-scan>

        <!-- 配置视图解析器 -->
        <bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
            <!-- 文件路径 -->
            <property name="prefix" value="/WEB-INF/pages/"/>
            <!-- 文件后缀 -->
            <property name="suffix" value=".jsp"/>
        </bean>

        <!-- 过滤静态资源 -->
        <mvc:resources mapping="/js/**" location="/js/"/>

        <!-- 开启mvc注解支持 -->
        <mvc:annotation-driven/>
    </beans>
    ```