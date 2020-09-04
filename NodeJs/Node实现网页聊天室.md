# websocket+nodejs实现聊天室
## 一、websocket
### 1、websocket简介
传统http协议，是基于请求和响应的，无法直接做到客户端向客户端发送消息。
websocket协议是基于tcp的一种新的网络协议。实现了浏览器与服务器全双工通信。全双工：客户端可以主动给服务器发送消息，服务器也可以主动给客户端发送消息。
websocket是一种持久协议，http是非持久的
传统http协议实现即使聊天，必须通过ajax轮询，就是客户端会一直向服务器发送请求来确认是否有消息，浪费性能和资源。

![image-20200813173335719](http://picture.youyouluming.cn/image-20200813173335719.png)

### 2、使用websocket
#### 2.1、在H5中使用websocket
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div {
            width: 200px;
            height: 200px;
            border: 1px solid #000;
        }
    </style>
</head>
<body>
<input type="text" placeholder="请输入">
<button>发送</button>

<!-- 显示内容 -->
<div></div>

<!-- websocket在浏览器的使用 H5提供了websocket的API-->
<script>
    let input = document.querySelector('input');
    let button = document.querySelector('button');
    let div = document.querySelector('div');

    //创建WebSocket('WebSocket服务器地址')
    let socket = new WebSocket('ws://echo.websocket.org');
    //监听WebSocket事件 open和WebSocket服务器连接成功触发
    socket.addEventListener('open',()=>{
        div.innerHTML = '连接成功';
    });

    //给webSocket发送消息
    button.addEventListener('click',()=>{
        let value = input.value;
        socket.send(value);
    });

    //接受websocket服务的消息
    socket.addEventListener('message',(msg)=>{
        console.log(msg.data);
        //把消息显示到div
        div.innerHTML = msg.data;
    });
    //端口服务
    socket.addEventListener('close',()=>{
        console.log('服务断开');
    });
</script>
</body>
</html>
```

#### 2.2、使用nodejs开发websocket
使用nodejs开发websocket需要一个依赖包 Nodejs Webs
> npm i nodejs-websocket

app.js代码
```js
const ws = require('nodejs-websocket');
const PORT = 3000

//创建server,每次只要有用户连接，回调执行就会给用户创建一个connect对象
const server = ws.createServer(connect => {
    console.log('用户连接成功');
    //用户传来数据，触发text事件
    connect.on('text', data => {
        console.log(`接受到用户的数据:${data}`);
        //接受到数据后给用户响应数据
        connect.sendText(data);
    });

    //连接关闭触发close事件
    connect.on('close',()=>{
        console.log('连接断开');
    });

    //注册error事件,用户端口后就会触发该异常
    connect.on('error',()=>{
        console.log('用户连接异常');
    });
});

server.listen(PORT, () => {
    console.log('监听3000');
});
```
把html的websocket服务器地址改为'ws://localhost:3000/'

运行效果

![image-20200813173457388](http://picture.youyouluming.cn/image-20200813173457388.png)
控制台
![image-20200813173512424](http://picture.youyouluming.cn/image-20200813173512424.png)


### 3、使用websocket开发一个简单的聊天室
app.js
```js
const ws = require('nodejs-websocket');

//记录当前连接的用户数量
let count = 0;

const server = ws.createServer(conn => {
    console.log('新连接');
    count++;
    //给用户一个固定的名字
    conn.userName = `用户${count}`;
    //告诉所有用户，有人加入聊天室
    broadcast(`${conn.userName}加入聊天室`);

    //接受到客户端的数据触发该事件
    conn.on('text', data => {
        //接受到某个用户的数据，告诉所有的用户此消息，广播
        broadcast(data);
    });
    //关闭连接
    conn.on('close', () => {
        console.log('关闭连接')
        count--;
        //有人退出也告诉所有的用户
        broadcast(`${conn.userName}离开了聊天室`)
    });
    //发送异常
    conn.on('error', () => {
        console.log('异常');
    });
});

//广播
const broadcast = (msg) => {
    //server.connection表示所有的用户
    server.connections.forEach(item => {
        //遍历出每个用户，挨个发消息
        item.send(msg);
    });
}

server.listen(3000, () => {
    console.log('监听3000');
});
```

index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        /*div {
            width: 200px;
            height: 200px;
            border: 1px solid #000;
        }*/
    </style>
</head>
<body>
<input type="text" placeholder="请输入">
<button>发送</button>

<!-- 显示内容 -->
<div></div>

<!-- websocket在浏览器的使用 H5提供了websocket的API-->
<script>
    let input = document.querySelector('input');
    let button = document.querySelector('button');
    let div = document.querySelector('div');

    //创建WebSocket('WebSocket服务器地址')
    let socket = new WebSocket('ws://localhost:3000/');
    //监听WebSocket事件 open和WebSocket服务器连接成功触发
    socket.addEventListener('open',()=>{
        div.innerHTML = '连接成功';
    });

    //给webSocket发送消息
    button.addEventListener('click',()=>{
        let value = input.value;
        socket.send(value);
        input.value = '';
    });

    //接受websocket服务的消息
    socket.addEventListener('message',(msg)=>{
        console.log(msg.data);
        //把消息显示到div,以追加的方式
        let dv = document.createElement('div');
        dv.innerHTML = msg.data;
        div.appendChild(dv);
    });
    //端口服务
    socket.addEventListener('close',()=>{
        console.log('服务断开');
    });
</script>
</body>
</html>
```

运行效果
![image-20200813173545138](http://picture.youyouluming.cn/image-20200813173545138.png)

#### 3.1、优化聊天室消息效果
把消息优化为一个对象：
* type：消息的类型
    * 0：表示进入聊天室的消息
    * 1：表示离开聊天室的消息
    * 2：正常聊天  
* msg：消息的内容
* time：聊天的具体时间
* 注意：发送的时候要把这个对象转换为JSON格式

app.js
```js
const ws = require('nodejs-websocket');
//进入消息
const TYPE_ENTER = 0;
//离开消息
const TYPE_LEAVE = 1;
//正常消息
const TYPE_MSG = 2;

//记录当前连接的用户数量
let count = 0;

const server = ws.createServer(conn => {
    console.log('新连接');
    count++;
    //给用户一个固定的名字
    conn.userName = `用户${count}`;
    //告诉所有用户，有人加入聊天室
    broadcast({
        type: TYPE_ENTER,
        msg: `${conn.userName}加入聊天室`,
        time: new Date().toLocaleDateString()
    });

    //接受到客户端的数据触发该事件
    conn.on('text', data => {
        //接受到某个用户的数据，告诉所有的用户此消息，广播
        broadcast({
            type: TYPE_MSG,
            msg: data,
            time: new Date().toLocaleDateString()
        });
    });
    //关闭连接
    conn.on('close', () => {
        console.log('关闭连接')
        count--;
        //有人退出也告诉所有的用户
        broadcast({
            type: TYPE_LEAVE,
            msg:  `${conn.userName}离开了聊天室`,
            time: new Date().toLocaleDateString()
        })
    });
    //发送异常
    conn.on('error', () => {
        console.log('异常');
    });
});

//广播
const broadcast = (msg) => {
    //server.connection表示所有的用户
    server.connections.forEach(item => {
        //遍历出每个用户，挨个发消息
        item.send(JSON.stringify(msg));
    });
}

server.listen(3000, () => {
    console.log('监听3000');
});
```

html的改变
```js
 const TYPE_ENTER = 0;
 const TYPE_LEAVE = 1;
 const TYPE_MSG = 2;
//接受websocket服务的消息
    socket.addEventListener('message',(e)=>{
        let data = JSON.parse(e.data);
        //把消息显示到div,以追加的方式
        let dv = document.createElement('div');
        //为不同的消息类型添加效果
        dv.innerHTML = data.msg + '---' + data.time;
        if (data.type === TYPE_ENTER) {
            dv.style.color = 'green';
        } else if (data.type === TYPE_LEAVE){
            dv.style.color = 'red';
        } else {
            dv.style.color = 'blue';
        }
        div.appendChild(dv);
    });
```

运行效果
![image-20200813173558031](http://picture.youyouluming.cn/image-20200813173558031.png)

## 二、socketio基本使用
### 2.1、无框架
Socket.IO是一个库，用于在浏览器和服务器之间实现实时、双向和基于事件的通信。
Socket.IO不是WebSocket 的实现。
安装socketio
>npm i --save socket.io

app.js
```js
//创建服务器
const http = require('http');
const fs = require('fs');
const app = http.createServer();


//监听request事件，请求服务时返回index.html
app.on('request', (req, res) => {
    fs.readFile(__dirname + '/index.html',
        (err, data) => {
            if (err) {
                res.writeHead(500);
                return res.end('Error loading index.html');
            }

            res.writeHead(200);
            res.end(data);
        });
})

app.listen(3000, () => {
    console.log('监听3000');
});

//socketio依赖于nodejs服务器
const io = require('socket.io')(app);
//监听用户连接的事件
//socket表示用户的连接
//socket emit表示触发某个事件   如果向浏览器发送一个数据，需要触发浏览器注册的某个事件
//socket on表示注册某个事件，如果需要获取浏览器数据，就需要注册一个事件，等待浏览器触发
io.on('connection', socket => {
    console.log('新用户连接');
    //给浏览器发送数据emit('发送的事件','发送的事件')
    socket.emit('send', {name: 'jack'});

    //获取浏览器发送的数据,注册事件只要和触发事件一样就行
    socket.on('clientData',data=>{
        console.log(data);
    })
});
```

index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
hello
<script src="/socket.io/socket.io.js"></script>
<script>
    //连接socket服务
    let socket = io('http://localhost:3000');
    //浏览器注册服务端
    socket.on('send',data =>{
        console.log(data);
    });

    //向服务器发送数据
    socket.emit('clientData',{name:'jack'});
</script>
</body>
</html>
```

发送数据使用emit，接受数据使用on。浏览器或服务器给对方发送数据，只需要触发相应的事件。

### 2.2、基于express框架的socketio
安装express
>npm i express

app.js
```js
const app = require('express')();
const server = require('http').Server(app);
const io = require('socket.io')(server);

server.listen(3000,()=>{
    console.log('监听3000');
});

app.get('/', (req, res) => {
    res.sendFile(__dirname + '/index.html');
});

io.on('connection', (socket) => {

    socket.on('clientData', (data) => {
        console.log(data);
    });
});
```

index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
hello
<script src="/socket.io/socket.io.js"></script>
<script>
    let socket = io('http://localhost:3000');
        socket.emit('clientData', { name: 'jack' });
</script>
</body>
</html>
```

## 三、使用socketio开发聊天室
导入静态资源
目录结构
![image-20200813173611288](http://picture.youyouluming.cn/image-20200813173611288.png)

### 1、服务端搭建
app.js
```js
/*
    启动聊天室服务端的程序
 */
const app = require('express')();
const server = require('http').Server(app);
const io = require('socket.io')(server);

server.listen(3000,()=>{
    console.log('服务器启动成功');
});

//处理静态资源,把public目录设置为静态资源
app.use(require('express').static('public'));

app.get('/', (req, res) => {
    //重定向到首页
    res.redirect('/index.html');
});

io.on('connection', (socket) => {
   console.log('新用户连接');
});
```

### 2、登录功能
#### 2.1、登录功校验
js/index.js
```js
/*
    聊天室主要功能
 */

//连接socketio
let socket = io('http://localhost:3000');

/*
    登录功能
 */

//选择头像，在li上注册一个点击事件，加now类 addClass:添加一个类 siblings:移除其他li上的类
$('#login_avatar li').on('click', function () {
    $(this).addClass('now').siblings().removeClass('now');
});

//点击按钮登录
$('#loginBtn').on('click',function () {
    //获取用户名,去空格
    let username = $('#username').val().trim();
    if (!username) {
        alert('请输入用户名');
        return
    }
    //获取头像 li.now表选中的li   attr('src')拿到头像具体路径
    let avatar = $('#login_avatar li.now img').attr('src');

    //和服务器通信,把用户名和头像传输到服务器
    socket.emit('login',{
        username: username,
        avatar: avatar
    });
});

//监听登录失败的请求
socket.on('loginError',data=>{
    alert('登录失败,用户名已经存在');
});
//监听登录成功的请求
socket.on('loginSuccess',data=>{
    alert('登录成功');
});

```

app.js
```js
//保存所有登录过的用户
const users = [];
//监听用户连接
io.on('connection', (socket) => {
   console.log('新用户连接');
   //监听客户端发送的login请求
   socket.on('login',data=>{
       //判断是否重名,从数组中找是否有这个名字
       let user = users.find(item => item.username === data.username);
       if (user) {
           //该用户存在，登录失败
            socket.emit('loginError',{msg:'登录失败'});
       }else {
           //该用户不存在,登录成功    先把用户存到数组中
           users.push(data);
           socket.emit('loginSuccess',data);
       }
   });
});
```

#### 2.2、登录成功显示个人信息
js/index.js
```js
//监听登录成功的请求
socket.on('loginSuccess',data=>{
    //显示聊天框，隐藏聊天窗口  fadeOut,fadeIn淡出淡入效果
    $('.login_box').fadeOut();
    $('.container').fadeIn();
    //设置登录成功后的用户信息
    $('.avatar_url').attr('src',data.avatar);
    $('.user-list .username').text(data.username);
});
```

#### 2.3、广播新用户加入群聊
app.js
```js
//监听用户连接
io.on('connection', (socket) => {
   console.log('新用户连接');
   //监听客户端发送的login请求
   socket.on('login',data=>{
       //判断是否重名,从数组中找是否有这个名字
       let user = users.find(item => item.username === data.username);
       if (user) {
           //该用户存在，登录失败
            socket.emit('loginError',{msg:'登录失败'});
       }else {
           //该用户不存在,登录成功    先把用户存到数组中
           users.push(data);
           socket.emit('loginSuccess',data);

           //广播消息，有人加入到聊天室 io.emit广播事件
           io.emit('addUser',data);
       }
   });
});
```

js/index.js
```js
//监听新用户加入的请求
socket.on('addUser', data => {
    //添加一条系统消息
    $('.box-bd').append(
        `<div class="system">
                <p class="message_system">
                    <span class="content">${data.username}加入了群聊</span>
                </p>
            </div>`
    )
});
```

#### 2.4、显示用户列表和用户数量
app.js
```js
//监听用户连接
io.on('connection', (socket) => {
   console.log('新用户连接');
   //监听客户端发送的login请求
   socket.on('login',data=>{
       //判断是否重名,从数组中找是否有这个名字
       let user = users.find(item => item.username === data.username);
       if (user) {
           //该用户存在，登录失败
            socket.emit('loginError',{msg:'登录失败'});
       }else {
           //该用户不存在,登录成功    先把用户存到数组中
           users.push(data);
           socket.emit('loginSuccess',data);

           //广播消息，有人加入到聊天室 io.emit广播事件
           io.emit('addUser',data);

           //显示目前聊天室的人数
           io.emit('userList',users);
       }
   });
});
```

js/index.js
```js
/监听用户列表消息
socket.on('userList', data => {
    //把userList中的数据动态显示到左侧菜单
    $('.user-list ul').html('');    //每次循环完把列表设为空，否则会重复叠加
    data.forEach(item => {
        $('.user-list ul').append(
            `<li class="user">
                <div class="avatar"><img src="${item.avatar}" alt=""/></div>
                <div class="name">${item.username}</div>
            </li>`
        )
    }
)
    //显示用户人数
    $('#userCount').text(data.length);
});
```

### 3、离开聊天室功能
app.js
```js
//监听用户断开连接 disconnect断开连接事件
    socket.on('disconnect', () => {
        //把当前用户信息从users中删除掉,findIndex找到当前用户的下标
        let idx = users.findIndex(item => item.username === socket.username);
        //删除
        users.splice(idx,1);
        //广播有人离开聊天室
        io.emit('delUser',{
            username:socket.username,
            avatar: socket.avatar
        })

        //更新userList
        io.emit('userList', users);
    })
```

js/index.js
```js
//监听删除用户消息
socket.on('delUser', data => {
    //添加一条系统消息
    $('.box-bd').append(
        `<div class="system">
                <p class="message_system">
                    <span class="content">${data.username}离开了群聊</span>
                </p>
            </div>`
    )
});
```

### 4、聊天功能
#### 4.1、基本聊天功能实现
app.js
```js
//监听聊天信息
    socket.on('sendMessage',data=>{
        console.log(data);
        //广播消息,如有数据库在此把数据存储
        io.emit('receiveMessage',data);
    })
```

js/index.js
```js
//定义两个全局变量保存username和avatar
var username;
var avatar;

//登录成功后保存用户信息
 username = data.username;
    avatar = data.avatar;
//聊天功能
$('.btn-send').on('click', () => {
    //获取聊天内容
    let content = $('#content').val().trim();
    //制空聊天区
    $('#content').val('');
    //判断是否空数据
    if (!content) {
        return alert('请输入内容');
    }
    //发给服务器
    socket.emit('sendMessage', {
        msg: content,
        username: username,
        avatar: avatar
    })
})

//监听聊天消息
socket.on('receiveMessage', data => {
    //把接受到的消息显示在聊天窗口
    //判断消息是自己或别人的
    if (data.username === username) {
        //自己的消息
        $('.box-bd').append(
            ` <div class="message-box">
                <div class="my message">
                    <img class="avatar" src="${data.avatar}" alt=""/>
                    <div class="content">
                        <div class="bubble">
                            <div class="bubble_cont">${data.msg}</div>
                        </div>
                    </div>
                </div>
            </div>`
        );
    } else {
        //别人的消息
        $('.box-bd').append(
            `<div class="message-box">
                <div class="other message">
                    <img class="avatar" src="${data.avatar}" alt=""/>
                    <div class="content">
                        <div class="nickname">${data.username}</div>
                        <div class="bubble">
                            <div class="bubble_cont">${data.msg}</div>
                        </div>
                    </div>
                </div>
            </div>`
        );
    }


})
```

#### 4.2、在发送完消息自动滚动到底部
Element.scrollIntoView() 方法让当前的元素滚动到浏览器窗口的可视区域内。
* 如果为true，元素的顶端将和其所在滚动区的可视区域的顶端对齐。
* 如果为false，元素的底端将和其所在滚动区的可视区域的底端对齐。

js/index.js
```js
function scrollIntoView() {
    //使用scrollIntoView()滚动到底部,children(':last')表示找到最后一个子元素,get(0)获取元素的dom对象
    $('.box-bd').children(':last').get(0).scrollIntoView(false);
}
```
每个发消息的监听都使用scrollIntoView方法，包括系统通知

#### 4.3、发送图片
app.js
```js
//发送图片 change表示该文件的选择
$('#file').on('change', function () {
    //拿到上传的文件
    var file = this.files[0];
    //把文件发送的服务器，使用H5的功能fileReader,读取上传的文件
    var fr = new FileReader();
    fr.readAsDataURL(file);
    fr.onload = function () {
        socket.emit('sendImage', {
            username: username,
            avatar: avatar,
            img: fr.result
        })
    }
})

//监听图片信息
socket.on('receiveImage', data => {
    //把接受到的消息显示在聊天窗口
    //判断消息是自己或别人的
    if (data.username === username) {
        //自己的消息
        $('.box-bd').append(
            ` <div class="message-box">
                <div class="my message">
                    <img class="avatar" src="${data.avatar}" alt=""/>
                    <div class="content">
                        <div class="bubble">
                            <div class="bubble_cont">
                                <img src="${data.img}">
                            </div>
                        </div>
                    </div>
                </div>
            </div>`
        );
    } else {
        //别人的消息
        $('.box-bd').append(
            `<div class="message-box">
                <div class="other message">
                    <img class="avatar" src="${data.avatar}" alt=""/>
                    <div class="content">
                        <div class="nickname">${data.username}</div>
                        <div class="bubble">
                            <div class="bubble_cont">
                                <img src="${data.img}">
                            </div>
                        </div>
                    </div>
                </div>
            </div>`
        );
    }
    //等待图片加载完再滚动
    $('.box-bd img:last').on('load', function () {
        scrollIntoView();
    });

});
```

js/index.js
```js
 //接受图片信息
    socket.on('sendImage',data=>{
        //广播消息,如有数据库在此把数据存储
        io.emit('receiveImage',data);
    })
```

#### 4.3、发送表情
使用Jquery emoji 插件
引入
![image-20200813173629826](http://picture.youyouluming.cn/image-20200813173629826.png)


在index.html引入css
```html
<link rel="stylesheet" href="lib/jquery-mCustomScrollbar/css/jquery.mCustomScrollbar.min.css"/>
<link rel="stylesheet" href="lib/jquery-emoji/css/jquery.emoji.css"/>
```
引入js
```html
<script src="lib/jquery-mCustomScrollbar/script/jquery.mCustomScrollbar.min.js"></script>
<script src="lib/jquery-emoji/js/jquery.emoji.min.js"></script>
```

emoji()参数参考
![image-20200813173704417](http://picture.youyouluming.cn/image-20200813173704417.png)

js/index.js
```js
//发送表情
//初始化jQuery-emoji插件
$('.face').on('click', function () {
    $('#content').emoji({
        //触发表情的按钮
        button: '.face',
        showTab: false,
        animation: 'slide',
        position: 'topRight',
        icons: [{
            name: "QQ表情",
            path: "../lib/jquery-emoji/img/qq/",
            maxNum: 91,
            excludeNums: [41, 45, 54],
            file: ".gif"
        }]
    });
})
```

注意：把index.html输入内容区域的textarea类改为div类，否则不支持表情的显示，但是div本身不支持编辑内容，再加上contenteditable，index.html改完还要改一下index.js里获取内容的方式，用原来的val获取不到div的内容，改用html()

项目运行截图

![image-20200813173730149](http://picture.youyouluming.cn/image-20200813173730149.png)

GitHub地址：https://github.com/812467457/wechat.git