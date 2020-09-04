# Linux
## Linux的概述
1. Linux的概述
Linux是基于Unix的
Linux是一种自由和开放源码的操作系统，存在着许多不同的Linux版本，但它们都使用了Linux内核。Linux可安装在各种计算机硬件设备中，比如手机、平板电脑、路由器、台式计算机诞生于1991 年10 月5 日。是由芬兰赫尔辛基大学学生Linus Torvalds和后来加入的众多爱好者共同开发完成
2. Linux的历史：
Linux最初是由芬兰赫尔辛基大学学生Linus Torvalds由于自己不满意教学中使用的MINIX操作系统， 所以在1990年底由于个人爱好设计出了LINUX系统核心。后来发布于芬兰最大的ftp服务器上，用户可以免费下载，所以它的周边的程序越来越多，Linux本身也逐渐发展壮大起来，之后Linux在不到三年的时间里成为了一个功能完善，稳定可靠的操作系统.
3. Linux系统的应用
服务器系统Web应用服务器、数据库服务器、接口服务器、DNS、FTP等等； 
嵌入式系统路由器、防火墙、手机、PDA、IP 分享器、交换器、家电用品的微电脑控制器等等，
高性能运算、计算密集型应用Linux有强大的运算能力。
桌面应用系统
移动手持系统
4. Linux的版本
Linux的版本分为两种：内核版本和发行版本；内核版本是指在Linus领导下的内核小组开发维护的系统内核的版本号
5. Linux的主流版本
![620d595a4fcaed5efad25af9ff8b0eae.png](http://picture.youyouluming.cn/ScreenClip.png)

## Linux的安装
### 虚拟机的安装
1. 什么是虚拟机
虚拟机：使用一些软件模拟一台虚拟的电脑
虚拟软件：
VmWare  收费
VirtualBox  免费
2. 安装VmWare
3. CentOS的安装
### Linux的远程访问
1. 安装一个远程访问的软件:CRT
2. 连接Linux

### Linux 的目录结构
![9dd4d6baae00028962b50569ca6e4473.png](http://picture.youyouluming.cn/ScreenClip [2].png)
## Linux 的常用命令
1. 列出文件列表命令：ls ll
用来显示当前目录下的内容。配合参数的使用，能以不同的方式显示目录内容。格式：ls[参数] [文件名]。在Linux中以"."开头的都是隐藏文件
常用：
    * ls：显示文件名或目录名
    * ls -a：显示文件所有内容，包含隐藏文件
    * ls -l (缩写为ll)：列出来的结果详细，有时间，是否可读写等信息 
   
2. 切换目录命令：cd
使用cd app 切换到app目录。pwd查看当前所在目录的字符串形式
常用：
    * cd ..：退回到上一级目录
    * cd /app1/app2：切换多级目录
    * cd ~：切换到主目录
    * cd -：切换到上一个所在目录

3. 创建目录和移除目录：mkdir rmdir
常用
    * mkdir app：创建app目录
    * rmdir app：删除空的app目录
    * mkdir -p app1/app2：创建多级目录
   
4. 浏览文件：cat more less tail    
常用
    * cat app：查看app文件所有内容
    * more app：用于显示内容超过一个画面长度的情况，按空格显示下一个画面，回车显示下一行内容，q键退出查看
    * less app：和more功能相似，多了上下键可以翻滚功能
    * tail -10 app：查看app文件最后10行信息，多用于查看日志文件
    * tail -f app：动态查看文件，根据文件的变化而变化
   
5. 文件操作：
5.1 复制剪切：cp mv 
    * cp a.txt aaa/b.txt：复制app文件到aaa文件夹并重命名b.txt（可选）
    * mv b.txt /root/bbb：剪切b.txt到bbb目录下
5.2 删除：rm
    * rm bbb.txt：删除bbb.txt
    * rm bbb -r：删除bbb文件夹(-rf 不询问是否删除)
    * rm -rf a：不询问递归删除
    * rm -rf * ：删除所有文件
    * rm -rf /* ：自杀
5.3 打包：tar
    * tar –cvf xxx.tar ./* ：打包
    * tar –zcvf xxx.tar.gz ./* ：打包并压缩
    * tar –xvf xxx.tar ：解压
    * tar -zxvf xxx.tar.gz -C /usr/aaa：将解压的结果放到某个目录
5.4 查找文件：find
    * find / -xxx：在根目录查找
    * find / -name catal*.log：条件查找
5.5 查找文件里符合条件的字符串
    * grep lang anaconda-ks.cfg：在文件中查找lang
    * grep lang anaconda-ks.cfg –color：高亮显示
    * grep lang anaconda-ks.cfg –color -A5 -B5：显示前五行后五行
   
6. 其他常用命令
pwd：显示当前所在目录
touch：创建一个空文件
clear/ctrl+L：清屏

## Vi和Vim编辑器
1. vim编辑器
使用vim a.txt 进入文件编辑。三种模式：命令行模式、插入模式、低行模式。
    切换到命令行模式：按Esc键；
    切换到插入模式：按 i 、o、a键；
    i 在当前位置前插入
    I 在当前行首插入
    a 在当前位置后插入
    A 在当前行尾插入
    o 在当前行之后插入一行
    O 在当前行之前插入一行
    切换到底行模式：按 :（冒号）
    打开文件：vim file
    退出：esc -> :q
    修改文件：输入i进入插入模式
    保存并退出：esc->:wq
    不保存退出：esc->:q!
快捷键：
    dd – 快速删除一行
    yy - 复制当前行
    nyy - 从当前行向后复制几行
    p - 粘贴
    R – 替换
2. 重定向输出>和>>
   * cat a.txt > b.txt：把a.txt的内容覆盖到b.txt
   * cat a.txt >> b.txt：把a.txt的内容追加到b.
3. 系统管理命令
    * ps -ef：查看所有进程
    * ps -ef | grep xxx：查找某一进程
    * kill 1234：杀掉1234编号进程
    * kil -9 1234：强制杀掉进程
4. 管道（作用是将一个命令的输出用作另一个命令的输入）
    * ls -- help | more：分页查看帮助信息
    * ps -ef | grep java：查询名称中包含Java的进程

## Linux 的权限命令
1. 文件权限
![5c8a675eef2c73314779ae5d580f70a8.png](http://picture.youyouluming.cn/ScreenClip [3].png)
![31f0b37dc4edac1812ff4b1b7f06f1cf.png](http://picture.youyouluming.cn/ScreenClip [4].png)
1.1 代表文件类型
   
    * -表示文件
    * d表示文件夹
    * l表示链接

   
    1.2 代表用户具有该文件的权限    
    * r：read 读        4
    * w：write 写      2
    * x：excute 执行  1
   
    1.3 当前组内其他用户具有该文件的权限
    * r：read 读
    * w：write 写
 * x：excute 执行    
   
    1.4 其他组内其他用户具有该文件的权限
    * r：read 读
    * w：write 写
    * x：excute 执行
   
2. 文件权限管理
    * chmod：变更文件或目录的权限 
    * chmod u=rwx,g=rx,o=rx a.txt：user用户具有rwx权限，group用户具有rx权限，其他用户具有rx权限。
    * chmod 755 a.txt：简写版

## Linux 上常用网络操作
1. 主机名配置
    * hostname 查看主机名
    * hostname xxx 修改主机名 重启后无效
    * 如果想要永久生效，可以修改/etc/sysconfig/network文件
  
2. ip地址配置
    * ifconfig 查看(修改)ip地址(重启后无效)
    * ifconfig eth0 192.168.12.22 修改ip地址
    如果想要永久生效
    * 修改 /etc/sysconfig/network-scripts/ifcfg-eth0文件
    * DEVICE=eth0 #网卡名称BOOTPROTO=static #获取ip的方式(static/dhcp/bootp/none)
    * HWADDR=00:0C:29:B5:B2:69 #MAC地址
    * IPADDR=192.168.177.129 #IP地址
    * NETMASK=255.255.255.0 #子网掩码
    * NETWORK=192.168.177.0 #网络地址
    * BROADCAST=192.168.0.255 #广播地址
    * NBOOT=yes #  系统启动时是否设置此网络接口，设置为yes时，系统启动时激活此设备。

3. 域名映射
    /etc/hosts文件用于在通过主机名进行访问时做ip地址解析之用,相当于windows系统的C:\Windows\System32\drivers\etc\hosts文件的功能

4. 网络服务管理 
    * service network status 查看指定服务的状态
    * service network stop 停止指定服务
    * service network start 启动指定服务
    * service network restart 重启指定服务
    * service --status–all 查看系统中所有后台服务
    * netstat –nltp 查看系统中网络进程的端口监听情况
    防火墙设置
    * 防火墙根据配置文件/etc/sysconfig/iptables来控制本机的”出”、”入”网络访问行为。
    * service iptables status 查看防火墙状态
    * service iptables stop 关闭防火墙
    * service iptables start 启动防火墙
    * chkconfig  iptables off 禁止防火墙自启

## Linux 上软件安装
* Linux上的软件安装有以下几种常见方式介绍
    1. 二进制发布包
软件已经针对具体平台编译打包发布，只要解压，修改配置即可
    2. RPM包
软件已经按照redhat的包管理工具规范RPM进行打包发布，需要获取到相应的软件RPM发布包，然后用RPM命令进行安装
    3. Yum在线安装
软件已经以RPM规范打包，但发布在了网络上的一些服务器上，可用yum在线安装服务器上的rpm软件，并且会自动解决软件安装过程中的库依赖问题
    4. 源码编译安装
软件以源码工程的形式发布，需要获取到源码工程后用相应开发工具进行编译打包部署。

* 上传与下载工具介绍
    1. FileZilla
    2. lrzsz
    我们可以使用yum安装方式安装 yum install lrzsz
    注意：必须有网络
    可以在crt中设置上传与下载目录
    上传：rz
    下载：sz 文件名
    3. SFTP
    alt + p 进入，put 路径+文件名

* 在Linux上安装JDK
    1. 上传JDK到Linux的服务器
    上传JDK
    卸载open-JDK
    java –version
    rpm -qa | grep java
    rpm -e --nodeps java-1.6.0-openjdk-1.6.0.35-1.13.7.1.el6_6.i686
    rpm -e --nodeps java-1.7.0-openjdk-1.7.0.79-2.5.5.4.el6.i686
    2. 在Linux服务器上安装JDK
    通常将软件安装到/usr/local
    直接解压就可以
    tar –xvf  jdk.tar.gz  -C 目标路径
    3. 配置JDK的环境变量
    3.1 vi /etc/profile
    3.2 在末尾行添加
    #set java environment
 JAVA_HOME=/usr/local/jdk/jdk1.7.0_71
 CLASSPATH=.:$JAVA_HOME/lib.tools.jar
    PATH=$JAVA_HOME/bin:$PATH
    export JAVA_HOME CLASSPATH PATH
保存退出
    3.3 source /etc/profile  使更改的配置立即生效
    
* 在Linux上安装Mysql
    1. mysql的安装文件上传到Linux的服务器
    将MySQL的tar解压，系统自带的MySQL删除
    2. 安装MYSQL服务端
    会自动生成一个密码，保存在/root/.mysql_secret，第一次登陆需要修改密码
    3. 安装MYSQL客户端
    rpm -ivh MySQL-client-5..6.22-1.el6.i686.rpm
    开启mysql服务
    service mysql start
    设置用户密码
    set password = password('root');
    4. Mysql服务加入到系统服务并自动启动操作
    chkconfig --add mysql
    自动启动：
    chkconfig mysql on
    查询列表：
    chkconfig
    5. 关于mysql远程访问设置
    设置远程访问权限
    grant all privileges on *.* to 'root' @'%' identified by 'root';
    刷新
    flush privileges;
    关闭防火墙
    service iptables stop;

* 在Linux上安装tomcat
    1. Tomcat上传到linux上
    2. 将上传的tomcat解压
    3. 在tomcat/bin目录下执行 startup.sh（注意防火墙）
    4. 查看目标 tomcat/logs/catalina.out

* 在Linux上安装redis
    1. 安装gcc-c++
    yum install gcc-c++
    2.  下载redis
    wget http://download.redis.io/releases/redis-3.0.4.tar.gz
    2.1 解压
         tar -xzvf redis-3.0.4.tar.gz
    2.2 编译安装
        切换至程序目录，并执行make命编译：
    cd redis-3.0.4 
    make
    执行安装命令
    make PREFIX=/usr/local/redis install

    3. 配置redis
    3.1 复制配置文件          到/usr/local/redis/bin目录
    cd redis-3.0.4
    cp redis.conf /usr/local/redis/bin
    4. 启动redis
    4.1 进入redis/bin目录
    cd redis/bin
    启动redis服务端
    ./redis-server redis.conf
    4.2  克隆新窗口，启动redis客户端
    ./redis-cli