# SVN

## 一、简介

SVN是一个开源的版本控制工具，把数据放在一个中央仓库的服务器，在开发时从中央仓库获取数据，开发完提交到中央仓库，在多人共同开发时，SVN可以合并所有人的数据，并且记录每一次提交的版本、人员信息和时间记录，有利于多人协作开发，并且方便回退到之前的版本。

## 二、基本使用

### 1、安装

注意：安装路径不要有空格和汉字

### 2、准备仓库

* 创建一个文件夹`D:\my_java\SVNRepositories`作为仓库。

* 初始化仓库cmd命令`svnadmin create D:\my_java\SVNRepositories`
* 生成的目录结构

![image-20200805164811633](http://picture.youyouluming.cn/image-20200805164811633.png)

* 启动SVN服务，指定一个版本库根目录`svnserve -d -r D:\my_java\SVNRepositories`，默认端口号3690.
* 把SVN服务注册到window的服务`sc create MySVNService binpath= "D:\my_java\VisualSVN Server\bin\svnserve.exe --service -r D:\my_java\SVNRepositories" start= auto depend= Tcpip`，第一个路径是SVN安装路径，第二个是SVN仓库路径。

### 4、在eclipse中使用SVN

#### 4.1、导入SVN的插件

把插件复制到eclipse的dropins目录下

![image-20200805174919224](http://picture.youyouluming.cn/image-20200805174919224.png)

在eclipse设置中搜索svn

![image-20200805175000228](http://picture.youyouluming.cn/image-20200805175000228.png)

表示成功

#### 4.2、新建一个svn的连接

先把SVN的面板调出来

![image-20200805175049766](http://picture.youyouluming.cn/image-20200805175049766.png)

然后新建一个连接

![image-20200805175108593](http://picture.youyouluming.cn/image-20200805175108593.png)

指定仓库路径

![image-20200805175230530](http://picture.youyouluming.cn/image-20200805175230530.png)

#### 4.3、上传项目

第一步

![image-20200805175747270](http://picture.youyouluming.cn/image-20200805175747270.png)

第二步

![image-20200805175803168](http://picture.youyouluming.cn/image-20200805175803168.png)

可能出现权限不够的错误，需要去仓库的配置文件的`svnserve.conf`下修改权限，修改内容为`anon-access = write`

导出后仓库就可以看见该项目

![image-20200805195717141](http://picture.youyouluming.cn/image-20200805195717141.png)

#### 4.4、拉取仓库中的项目

![image-20200805195807334](http://picture.youyouluming.cn/image-20200805195807334.png)

#### 4.5、提交项目

![image-20200805203440421](http://picture.youyouluming.cn/image-20200805203440421.png)

如果在多人同时操作同一处的代码时，会出现冲突导致无法提交，这时需要先解决冲突，然后标记为已解决。



### 5、SVN权限管理

上面的操作无需任何登录操作，直接就可以访问，实际开发肯定不会这样，需要对不同的人有不同的用户密码和权限。

在`svnserve.conf`中配置，把之前的`anon-access = write`注释上，放开`auth-access = write`表示开启登录认证，放开`authz-db = authz`表示开启权限控制，再把`password-db = passwd`放开表示会去读取passwd中的用户信息，接着去`passwd`文件中配置用户名和密码，格式如：`sally = sallyssecret`。去`authz`文件配置权限。

配置权限详情

```properties
[groups]
# harry_and_sally = harry,sally
# harry_sally_and_joe = harry,sally,&joe
# 配置分组
dev = root,jack
test = tom

# [/foo/bar]
# harry = rw
# &joe = r
# * =

# [repository:/baz/fuz]
# @harry_and_sally = rw
# * = r
# 配置权限  给组分配权限，从根目录
[/]
@dev=rw
@test=r
# 其他人无权限的配置方法
*=
```



### 6、SVN锁操作

![image-20200805211404325](http://picture.youyouluming.cn/image-20200805211404325.png)

## 三、TortoiseSVN

TortoiseSVN是SVN的独立客户端