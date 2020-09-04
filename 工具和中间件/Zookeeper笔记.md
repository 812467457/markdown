# Zookeeper

## 一、Zookeeper简介

Zookeeper是一个高性能分布式协调技术，开源的分布式系统协调服务。是一个为分布式程序一致性和分布式协调技术技术服务的软件。去协调分布式系统中各个模板的技术。

从设计模式来说，Zookeeper是一个基于观察者模式的分布式服务管理框架，负责存储和管理各个模块的数据，然后接受观察者的注册，一旦这些数据状态发生变化，Zookeeper就负责通知已经在Zookeeper上注册的那些观察者做出相应的反应。

Zookeeper = 类似uniz文件系统 + 通知机制+ znode节点



## 二、Zookeeper安装

* 把Zookeeper的安装包拷贝到Linux下
* 解压`tar -zxvf zookeeper-3.4.11.tar.gz`
* 创建Zookeeper的目录`mkdir /myzookeeper`，把刚才解压的内容的拷贝到该目录`cp -r zookeeper-3.4.11 /myzookeeper/`
* 在conf文件夹下把`zoo_sample.cfg`拷贝为`zoo.cfg`：`cp zoo_sample.cfg zoo.cfg`
* 需要Java环境
* 启动服务：`./zkServer.sh start`，使用Zookeeper自带的查询方法查询是否启动成功`echo ruok | nc 127.0.0.1 2181`，返回imok表示启动成功。
* 连接客户端：`./zkCli.sh`
* 关闭服务：`./zkServer.sh stop`

## 三、Zookeeper入门使用

### 1、常用命令

* 查看当前znode中包含的内容：`ls /`
* 查看当前znode中包含的内容和状态信息：`ls2 /`
* 显示节点信息：`stat /`

* 创建节点和数据：`create /yylm demo01`
* 创建持久化节点和数据（自动分配序号，防止重名）：`create -s /yylm data`
* 创建临时目录节点（和服务端断开，该节点就销毁）：`create -e /yylm data`
* 获取节点数据：`get /yylm`
* 更新节点数据：`set /yylm demo02`
* 删除无子节点的目录：`delete`
* 删除带子节点的目录：`rmr`

### 2、四字命令

四字命令固定用法`echo 四字命令| nc 127.0.0.1 2181`

![image-20200810214034725](http://picture.youyouluming.cn/image-20200810214034725.png)

### 3、Stat结构体

![image-20200810172021187](http://picture.youyouluming.cn/image-20200810172021187.png)

Zookeeper内部维护了一套类似于Unix的树形结构：由znode组成的集合。znode=path+data+Stat

## 四、Java调用Zookeeper

### 1、需要的依赖

```xml
<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.11</version>
</dependency>
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.6.1</version>
</dependency>
```

### 2、代码

```Java
public class HelloZkp {
    private static final String CONNECTSTRING = "Linux主机地址:2181";
    private static final String PATH = "/yylm01";
    private static final int SESSION_TIMEOUT = 20 * 1000;

    /**
     * 新建ZK连接
     *
     * @return
     * @throws IOException
     */
    public ZooKeeper startZK() throws IOException {
        return new ZooKeeper(CONNECTSTRING, SESSION_TIMEOUT, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {

            }
        });
    }

    /**
     * 创建ZNode节点
     *
     * @param zk
     * @param path
     * @param data
     * @throws KeeperException
     * @throws InterruptedException
     */
    public void createZNode(ZooKeeper zk, String path, String data) throws KeeperException, InterruptedException {
        //节点路径，节点数据，节点权限，节点类型（持久化、临时....）
        zk.create(path, data.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    }


    /**
     * 获取
     *
     * @param zk
     * @param path
     * @return
     * @throws KeeperException
     * @throws InterruptedException
     */
    public String getZookeeper(ZooKeeper zk, String path) throws KeeperException, InterruptedException {
        String result = "";
        //路径，观察者，校验状态
        byte[] data = zk.getData(path, false, new Stat());
        result = new String(data);
        return result;
    }

    /**
     * 关闭连接
     *
     * @param zk
     * @throws InterruptedException
     */
    public void stopZK(ZooKeeper zk) throws InterruptedException {
        if (zk != null) {
            zk.close();
        }
    }
}
```

## 五、Zookeeper通知机制

### 1、简介

客户端监听它所关系的目录节点，当目录节点发生变化时，Zookeeper会通知客户端。

### 2、观察者（watch）

客户端可以在每个ZNode节点上设置一个观察，如果被观察服务端的ZNode节点有变化，那么watch就会触发，这个watch所属的客户端将接收到一个通知包被告知节点已经发生变化，把相应的通知事件告知给设置过watch的客户端端。

可以设置watch的操作：getData()，getChildren()，和exists()。

watch特点：

* 一次触发：数据变化向客户端发送一个watch，是一次性的动作。
* 发向客户端：watches是异步向客户端发消息的
* 设置watch：所有的’写‘请求都会通知





