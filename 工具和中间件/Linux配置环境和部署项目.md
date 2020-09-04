# Linux配置环境和部署项目

## 一、安装JDK

### 1、解压安装包

`tar -zxvf jdk-8u121-linux-x64.tar.gz`

### 2、配置环境变量

打开`vim /etc/profile`

在最低端加入内容

```properties
JAVA_HOME=刚才解压后jdk的文件夹
PATH=$PATH:$JAVA_HOME/bin
export JAVA_HOME PATH
```

退出vim

`esc + : + wq`

重新加载配置文件

`source /etc/profile`

检查是否安装成功

`java -version`



## 二、安装tomcat

### 1、解压

`tar -zxvf apache-tomcat-7.0.75.tar.gz`

### 2、启动和停止

启动服务

`./startup.sh `

测试

http://主机ip地址:8080/	注意：云服务器需要先去安全组放行端口

停止服务

`./shutdown.sh`

## 三、安装MySQL（5.7）

### 1、安装前的检查工作

* 检查MySQL：`rpm -qa|grep mysql`
  * 如果存在就卸载掉：`rpm -e --nodeps mysql-libs`

* 检查mariadb：`rpm -qa|grep mariadb`
  * 如果存在就卸载掉：`rpm -e --nodeps mariadb-libs`

* 检查是否存在ibaio：`rpm -qa|grep libaio`
  * 不存在就安装：`yum install libaio`
* 检查是否存在net-tools：`rpm -qa|grep net-tools`
  * 不存在就安装：`yum install net-tools`

### 2、安装（必须按照顺序按照）

1. `rpm -ivh mysql-community-common-5.7.16-1.el7.x86_64.rpm`
2. `rpm -ivh mysql-community-libs-5.7.16-1.el7.x86_64.rpm `
3. `rpm -ivh mysql-community-client-5.7.16-1.el7.x86_64.rpm`
4. `rpm -ivh mysql-community-server-5.7.16-1.el7.x86_64.rpm`

5. 查看MySQL版本：`mysqladmin --version`

### 3、初始化

MySQL5.7不会自动初始化，需要手动初始化

`mysqld --initialize --user=mysql`

登录密码是随机的，查看日志获取密码

`cat /var/log/mysqld.log`

![image-20200813123635555](http://picture.youyouluming.cn/image-20200813123635555.png)

### 4、启动

* 启动：`systemctl start mysqld`
* 检查启动是否成功：`systemctl status mysqld`
* 登录：`mysql -uroot -p`，然后输入刚才日志中的密码（注意：此时输入密码时看不见的，输完回车就行）
* 重置密码：`ALTER USER 'root'@'localhost' IDENTIFIED by '新密码';`
* 退出后重新使用新密码登录

### 5、设置字符集

* 打开配置文件：`vim /etc/my.cnf`
* 最后面增加一行：`character_set_server=utf8`
* 如果已经存在数据库和表，还要修改库和表的字符集：
  * 修改数据库字符集：`alter database mydb character set 'utf8';`
  * 修改表的字符集：`alter table mytb convert to character set 'utf8';`

### 6、权限设置

* 创建新的用户：跳转到mysql数据库下，创建一个用户用于远程连接`create user root identified by 'admin';`
* 给用户授权：`grant all privileges on *.*to root@'%' identified by 'admin';`

## 四、安装Redis

### 1、安装gcc-c++

`yum install -y gcc-c++`

### 2、解压

`tar -zxvf redis-4.0.2.tar.gz`

### 3、编译

先去`vim src/Makefile`修改目录

![image-20200813155426931](http://picture.youyouluming.cn/image-20200813155426931.png)

在解压后的文件夹中编译

`make`

安装

`make install`

### 4、配置

把配置文件赋值到llocal下

`cp /opt/software/redis-4.0.2/redis.conf  /usr/local/redis/`

修改配置文件

`vim /usr/local/redis/redis.conf`

修改一

![image-20200813161259896](http://picture.youyouluming.cn/image-20200813161259896.png)

修改二 

![image-20200813161402921](http://picture.youyouluming.cn/image-20200813161402921.png)

修改三

![image-20200813161439477](http://picture.youyouluming.cn/image-20200813161439477.png)

保存退出

创建修改二中的目录`mkdir /var/redis`

### 5、启动

服务端

`/usr/local/redis/bin/redis-server /usr/local/redis/redis.conf`

客户端

`/usr/local/redis/bin/redis-cli`

## 五、部署项目（springBoot项目为例）

把项目打包为jar包上传到服务器上

然后运行nohup java -jar jar包的名字，再通过本机公网ip+项目端口号就可以访问到项目了