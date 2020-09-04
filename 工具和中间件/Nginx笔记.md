# Nginx

## 一、Nginx简介

Nginx是一个高性能的HTTP和反向代理的服务器，特点是占内存少，并发能力强。Nginx虽然是一个服务器，但是不支持作为Java的服务器，Java程序使用Nginx只能通过tomcat配合完成。

Nginx功能：

* 反向代理：客户端直接访问到的服务器不是项目真正所在的服务器，使用nginx对客户端的请求进行反向代理到真正的服务器上，保护服务器的安全
* 负载均衡：使用nginx把请求合理的分配到每一台服务器，防止某一台服务器压力过大。根据客户端访问ip对其就近分配服务器。
* 动静分离：文件资源和项目分在不同的服务器，使用neinx访问，减少项目的容量，方便维护。

## 二、Nginx安装

### 1、第一步把所需要的包复制到Linux

![image-20200805235403985](http://picture.youyouluming.cn/image-20200805235403985.png)

###  2、第二步先安装PCRE

* 解压`tar -zxvf pcre-8.37.tar.gz`

* 进入pcre文件夹`cd pcre-8.37/`，执行configure`./configure`
* 执行编译`make`，执行安装`make install`

### 3、第三步安装penssl

* 解压`tar -zxvf openssl-1.0.1t.tar.gz`
* 进入poenssl文件夹`cd openssl-1.0.1t/`，执行config  `./config`
* 编译并安装`make && make install`

### 4、第四步安装zilb

* 解压`tar -zxvf zlib-1.2.8.tar.gz`
* 进入zilb文件夹`cd zlib-1.2.8/`，执行configure  `./configure`
* 编译并安装`make && make install`

### 5、第五步安装nginx

* 解压`tar -zxvf nginx-1.11.1.tar.gz `
* 进入nginx文件夹`cd nginx-1.11.1/`，执行configure  `./configure`
* 编译并安装`make && make install`

注意：编译过程中可能会遇到

![image-20200806003511675](http://picture.youyouluming.cn/image-20200806003511675.png)

解决：

* 使用`make CFLAGS='-Wno-implicit-fallthrough'`命令编译
* 修改`vim src/os/unix/ngx_user.c`下的，注释掉。

![](http://picture.youyouluming.cn/image-20200806003652201.png)

### 6、验证安装成功

* Nginx默认安装在 `cd /usr/local/`
* 运行`/usr/local/nginx/sbin` 下的`./nginx`，注意：默认80端口，修改端口号：在`/usr/local/nginx/conf`目录下的`nginx.conf`文件中。再次注意：如果是云服务器记得放开对应端口号。
* 启动：`./nginx`，访问：主机ip+端口号

* 检查是否启动成功`ps aux | grep nginx`

### 7、其他命令

* 启动：`./nginx`
* 关闭：`./nginx -s stop`
* 重新加载：`./nginx -s reload`，此命令为热加载，不会影响正在运行的服务

## 三、Nginx小案例

目标：准备一个web项目，使用nginx对其反向代理，访问一个定义好的域名代理到具体的服务器上。

项目代码，随机生成一个userid，返回userid和访问端口

```java
	public class HelloWorldServlet extends HttpServlet{
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		HttpSession session= request.getSession();
		int port2=request.getLocalPort();
		if(session.getAttribute("userid")==null){
			 String userid= String.valueOf(new Random().nextInt(100)) ;
			 session.setAttribute("userid", userid);
			 response.getWriter().append("Hello, "+userid+",this is "+port2+ " port");
		}else{
			String userid=(String)session.getAttribute("userid");
			response.getWriter().append("Welcome back, "+userid+", this is "+port2+" port") ;
		}
 
	}
}
```



negix配置，打开`/usr/local/nginx/`下的nginx.conf  `vim nginx.conf`，添加下面配置，weight表示权重，负载均衡的策略是轮询，会根据权重来分配。

```properties
 upstream myserver{
      # ip_hash;
      server 47.94.168.225:8080 weight=1;
      server 47.94.168.225:8081 weight=1;
  }

server {
    location / {   
    #反向代理路径
    proxy_pass http://myserver;

    #反向代理的超时时间
    proxy_connect_timeout 10;
    }
}    
```

注意：如果是虚拟机就从window系统查看VMware查看对应的ip地址，如果是服务器就写服务器的地址，注意配置一下tomcat的两个端口号。

效果：访问nginx，会自动把请求以轮询的方式交给两个不同的tomcat

![image-20200806161537046](http://picture.youyouluming.cn/image-20200806161537046.png)

再次访问

![image-20200806161559869](http://picture.youyouluming.cn/image-20200806161559869.png)

## 四、Nginx原理配置

### 1、原理

nginx启动后的进程

![image-20200806161917533](http://picture.youyouluming.cn/image-20200806161917533.png)

可以发现有一个master和两个worker，nginx是由用户启动master进程，然后master去配置文件读取worker配置并管理监控worker，配置几个worker就会启用几个，具体工作是由worker做的。在客户端请求过来时多个worker回去抢请求，谁抢到谁请求，并且多个worker之间相互独立，某个worker不能使用不影响其他worker，热加载就是将没在工作中的worker先重启，工作中的worker等请求结束再重启。因为worker会占用cup资源，所以worker配置的数量根据cpu决定。

### 2、配置

```properties
#安全问题，建议用nobody,不要用root.
#user  nobody;


#work绑定cpu(4 work绑定4cpu)
#worker_cpu_affinity 0001 0010 0100 1000;

#work绑定cpu (4 work绑定8cpu中的4个) 。
#worker_cpu_affinity 0000001 00000010 00000100 00001000;  



#error_log path(存放路径) level(日志等级)^Kpath表示日志路径，level表示日志等级，
#具体如下：^K[ debug | info | notice | warn | error | crit ]
#从左至右，日志详细程度逐级递减，即debug最详细，crit最少，默认为crit。 

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;


events {
    #当然，这里说的是最大连接数，对于HTTP请求本地资源来说，能够支持的最大并发数量是worker_connections * worker_processes，
    #如果是支持http1.1的浏览器每次访问要占两个连接，
    #所以普通的静态访问最大并发数是： worker_connections * worker_processes /2，
    #而如果是HTTP作为反向代理来说，最大并发数量应该是worker_connections * worker_processes/4。
    #因为作为反向代理服务器，每个并发会建立与客户端的连接和与后端服务的连接，会占用两个连接。

    worker_connections  1024;

    use epoll;

    # 默认是on ,开启nginx的抢占锁机制。
    accept_mutex  on;
}
    #如果 不能从mime.types找到映射的话，用以下作为默认值
    default_type  application/octet-stream;



     #日志位置
     # access_log  logs/host.access.log  main;

     #一条典型的accesslog：

     #1）101.226.166.254:(用户IP)
     #2）[21/Oct/2013:20:34:28 +0800]：(访问时间) 
     #3）GET：http请求方式，有GET和POST两种
     #4）/movie_cat.php?year=2013：当前访问的网页是动态网页，movie_cat.php即请求的后台接口，year=2013为具体接口的参数
     #5）200：服务状态，200表示正常，常见的还有，301永久重定向、4XX表示请求出错、5XX服务器内部错误
     #6）5209：传送字节数为5209，单位为byte
     #7）"http://www.baidu.com"：refer:即当前页面的上一个网页

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                       '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';



    #开启从磁盘直接到网络的文件传输，适用于有大文件上传下载的情况，提高IO效率。
    sendfile        on;


    #一个请求完成之后还要保持连接多久, 默认为0，表示完成请求后直接关闭连接。
    #keepalive_timeout  0;
    keepalive_timeout  65;



    #开启或者关闭gzip模块
    #gzip  on ;

    #设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。
    #gzip_min_lenth 1k;

    # gzip压缩比，1 压缩比最小处理速度最快，9 压缩比最大但处理最慢（传输快但比较消耗cpu）
    #gzip_comp_level 4;

    #匹配MIME类型进行压缩，（无论是否指定）"text/html"类型总是会被压缩的。
    #gzip_types types text/plain text/css application/json  application/x-javascript text/xml  



    #动静分离
    #服务器端静态资源缓存，最大缓存到内存中的文件，不活跃期限
    open_file_cache max=655350 inactive=20s;

    #活跃期限内最少使用的次数，否则视为不活跃。
    open_file_cache_min_uses 2;

    #验证缓存是否活跃的时间间隔
    open_file_cache_valid 30s;



    upstream myserver{

    # 1、轮询（默认）
    # 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
    # 2、指定权重
    # 指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
    #3、IP绑定 ip_hash
    # 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。
    #4、备机方式 backup
    # 正常情况不访问设定为backup的备机，只有当所有非备机全都宕机的情况下，服务才会进备机。
    #5、fair（第三方）
    #按后端服务器的响应时间来分配请求，响应时间短的优先分配。   
    #6、url_hash（第三方）
    #按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。


      # ip_hash;
             server 47.94.168.225:8080 weight=1;
             server 47.94.168.225:8081 weight=1;

      #fair

      #hash $request_uri
      #hash_method crc32

      }

    server {
        #监听端口号
        listen       8082;

        #服务名
        server_name  47.94.168.225;

        #字符集
        #charset utf-8;




        #location [=|~|~*|^~] /uri/ { … }   
        # = 精确匹配
        # ~ 正则匹配，区分大小写
        # ~* 正则匹配，不区分大小写
        # ^~  关闭正则匹配

        #匹配原则：

        # 1、所有匹配分两个阶段，第一个叫普通匹配，第二个叫正则匹配。
        # 2、普通匹配，首先通过“=”来匹配完全精确的location
        #   2.1、 如果没有精确匹配到， 那么按照最大前缀匹配的原则，来匹配location
        #   2.2、 如果匹配到的location有^~,则以此location为匹配最终结果，如果没有那么会把匹配的结果暂存，继续进行正则匹配。
        # 3、正则匹配，依次从上到下匹配前缀是~或~*的location, 一旦匹配成功一次，则立刻以此location为准，不再向下继续进行正则匹配。
        # 4、如果正则匹配都不成功，则继续使用之前暂存的普通匹配成功的location.


        location / {   # 匹配任何查询，因为所有请求都已 / 开头。但是正则表达式规则和长的块规则将被优先和查询匹配。

            #定义服务器的默认网站根目录位置
            root   html;

            #默认访问首页索引文件的名称
            index  index.html index.htm;

            #反向代理路径
            proxy_pass http://myserver;

            #反向代理的超时时间
            proxy_connect_timeout 10;

            proxy_redirect default;

         }

         location  /images/ {
            root images ;
         }

         location ^~ /images/jpg/ {  # 匹配任何已 /images/jpg/ 开头的任何查询并且停止搜索。任何正则表达式将不会被测试。 
            root images/jpg/ ;


         }
         location ~*.(gif|jpg|jpeg)$ {

              #所有静态文件直接读取硬盘
              root pic ;

              #expires定义用户浏览器缓存的时间为3天，如果静态页面不常更新，可以设置更长，这样可以节省带宽和缓解服务器的压力
              expires 3d; #缓存3天
         }


        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
}
```

