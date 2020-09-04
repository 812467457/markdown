

# Git

## 一、简介

Git是一个免费、开源的**分布式**版本控制系统，用于敏捷高效地处理任何或小或大的项目。Git 不仅仅是个版本控制系统，它也是个内容管理系统(CMS)，工作管理系统等。

## 二、安装

![image-20200807110224340](http://picture.youyouluming.cn/image-20200807110224340.png)

![image-20200807110259214](http://picture.youyouluming.cn/image-20200807110259214.png)

![image-20200807110313409](http://picture.youyouluming.cn/image-20200807110313409.png)

![image-20200807110340055](http://picture.youyouluming.cn/image-20200807110340055.png)

![image-20200807110513431](http://picture.youyouluming.cn/image-20200807110532142.png)

![image-20200807110555001](http://picture.youyouluming.cn/image-20200807110555001.png)

![image-20200807110616734](http://picture.youyouluming.cn/image-20200807110616734.png)

![image-20200807110651986](http://picture.youyouluming.cn/image-20200807110651986.png)

![image-20200807110710479](http://picture.youyouluming.cn/image-20200807110710479.png)

![image-20200807110723275](http://picture.youyouluming.cn/image-20200807110723275.png)

![image-20200807114456161](http://picture.youyouluming.cn/image-20200807114456161.png)

![image-20200807114505973](http://picture.youyouluming.cn/image-20200807114505973.png)

![image-20200807114515272](http://picture.youyouluming.cn/image-20200807114515272.png)

成功后

![image-20200807114727682](http://picture.youyouluming.cn/image-20200807114727682.png)

Git GUI

![image-20200807114749327](http://picture.youyouluming.cn/image-20200807114749327.png)

GitBash

![image-20200807114821983](http://picture.youyouluming.cn/image-20200807114821983.png)

初始化设置个人信息，作用于全局

> git config --global user.name "xxx"
>
> git config --global user.email "xxx"

## 三、Git的简单使用

### 1、创建版本仓库

在项目文件夹内执行` git init`，文件夹内的项目就已经被git管理了，文件夹内有一个.git文件夹，有整个项目的数据。

![image-20200807122314204](http://picture.youyouluming.cn/image-20200807122314204.png)

### 2、提交文件

* 将文件添加到暂存区：`git add a.txt`
* 提交到本地仓库：`git commit`，编写注释提交内容
* 或者`git commit -m "注释内容"`，直接带注释提交

### 3、查看文件提交记录

* 查看提交记录 ：`git log`
* 简易信息查看提交记录：`git log --pretty=oneline`

### 4、回退历史

* 回退到上一次提交：`git reset --hard HEAD^`
* 回退n步：`git reset --hard HEAD~n`  (n为回退的步数)

### 5、版本穿越

* 查看历史的版本号：`git reflog`
* 穿越到指定版本：`git reset --hard 14d1ed9`   （指定文件的版本号）

### 6、还原文件

* 用之前的版本覆盖当前版本` git checkout a.txt`

### 7、删除

* 删除原文件
* 提交 `git commit -m  ""`

## 四、Git的分支

### 1、创建分支

* 创建一个分支：`git branch 分支名` 
* 创建并进入分支：`git checkout -b 分支名`
* 查看分支：`git branch -v`

### 2、切换分支

* `git checkout "分支名"`

### 3、合并分支

* 先切换到主分支` git checkout master`
* 合并：`git merge "分支名"`

### 4、删除分支

* ` git branch -D test`

### 5、冲突

如果多人同时操作同一处代码并合并时git无法判断保留哪个版本，会出现冲突，需要手工判断解决冲突。

* 查看哪里出现冲突：`git diff`
* 然后进入具体的代码解决冲突
* 重新add、commit就解决完冲突了

## 五、GitHub

### 1、搭建代码仓库

* `git init`
* git config....

### 2、提交代码

* git add ....
* git commit 

### 3、准备Github

* 账号
* 搭建项目

### 4、把项目推送到远程服务器

* 增加远程地址：`git remote add origin https://github.com/xxxxx/xxxxx`
* 推送到远程仓库：`git push origin master`，输入GitHub用户名和密码

### 5、把项目从远程服务器取出

* 克隆一个项目：`git clone https://github.com/xxxxx/xxxxx.git 可以自定义名称`

### 6、把修改后的项目提交到服务器

* 先add、commit
* 提交：`git push origin master`

### 7、取出项目

* `git pull origin master`

* 如果向服务器提交数据时，出现冲突先pull一下，pull时出现冲突就使用上面分支解决冲突的方法

## 六、SSH方式

不必每次操作都输入用户密码

### 1、配置SSH Key

* 检查是否有ssh配置：`cd ~`，`cd .ssh/`，如果存在就删除该目录
* 创建SSH Key：`ssh-keygen -t rsa -C gitHub账号`
* 在./ssh文件下有一个公钥的文件，把内容复制到GitHub配置SSH Keys中
* 再次从项目中克隆就是要SSH协议的URL

### 2、测试连接

* 再增加一个远程地址：`git remote add originssh 项目SSH地址`
* `git push originssh master`

## 七、Eclipse操作Git

### 1、安装egit插件

![image-20200808003056302](http://picture.youyouluming.cn/image-20200808003056302.png)

### 2、操作

![image-20200808003154624](http://picture.youyouluming.cn/image-20200808003154624.png)

可以add和commit

![image-20200808003217796](http://picture.youyouluming.cn/image-20200808003217796.png)

commit界面

![image-20200808003309394](http://picture.youyouluming.cn/image-20200808003309394.png)

### 3、远程服务器操作

向服务器push整工程

![image-20200808093828025](http://picture.youyouluming.cn/image-20200808093828025.png)

![image-20200808093946341](http://picture.youyouluming.cn/image-20200808093946341.png)

### 4、把服务器上的项目导入eclipse

![image-20200808095727428](http://picture.youyouluming.cn/image-20200808095727428.png)

![image-20200808095745763](http://picture.youyouluming.cn/image-20200808095745763.png)

上传到GitHub时就只有单独的项目，所以导出到eclipse时也是只有单独的项目，不包含其他jar包和配置文件。

把项目改为动态工程

![image-20200808101251165](http://picture.youyouluming.cn/image-20200808101251165.png)

![image-20200808101339296](http://picture.youyouluming.cn/image-20200808101339296.png)

## 八、Git工作流

![image-20200808124643993](http://picture.youyouluming.cn/image-20200808124643993.png)

实际开发中会存在多个分支进行开发，最终合并到主分支。

* 主干分支：负责管理正在生产环境运行的代码，保持和生产环境代码一致
* 开发分支：管理正在开发过程中的代码
* bug修理分支：负责管理生产环境下出现的紧急修复代码，从主干分出，修改完合并到主干
* 版本发布分支：上线前最后阶段测试 
* 功能分支：开发各个模块的分支，开发完后合并到开发分支

### 1、Eclipse创建新的分支

![image-20200808141602956](http://picture.youyouluming.cn/image-20200808141602956.png)

创建完分支后正常提交

### 2、拉取分支

另一个人拉取之前他人提交的分支到本地查看

先pull一下

![image-20200808141740489](http://picture.youyouluming.cn/image-20200808141740489.png)

找到刚才的分支

### 3、合并分支到主支线

![image-20200808144100445](http://picture.youyouluming.cn/image-20200808144100445.png)

合并之后再次向服务器提交