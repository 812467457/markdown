





# Vue.js

## 一、概述

vue是一套用户界面的渐进式框架（可以使用该框架的某个或某些组件，不会要求整个系统都使用该框架），与其他框架不同的是，Vue被设计为可以自底向上逐层应用，Vue的核心库只关注视图层，容易上手，还便于与第三方库或已有项目做整合。另一方面，当与现代化工具链以及各种支持类库结合时，Vue也能够为复杂的单页提供驱动。

## 二、搭建Vue.js环境

### 1、安装Vue

三种方式

* 下载Vue的Js文件并引入
* 使用cdn方式引用
* 使用npm安装

### 2、使用Npm安装

创建一个静态工程，然后初始化该静态工程

```sh
npm init -y
```

安装vue

```sh
npm install vue --save
```

### 3、小demo

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<div id="app">
    {{message}}
</div>

<script src="../node_modules/vue/dist/vue.min.js"></script>
<script>
    let app = new Vue({
        //作用域
        el:'#app',
        //数据
        data:{
            message:"hello vue"
        }
    })
</script>
</body>
</html>
```

![image-20200813212607279](http://picture.youyouluming.cn/image-20200813212607279.png)

## 三、基本语法

### 1、基本数据渲染和指令

vue指令前缀是'v-'

比如

![image-20200813215452048](http://picture.youyouluming.cn/image-20200813215452048.png)

鼠标悬浮在标题就会有提示数据。

还可以直接使用语法糖省略写为一个':'

![image-20200813215606224](http://picture.youyouluming.cn/image-20200813215606224.png)

效果如上

### 2、双向数据绑定

使用v-model进行双向数据绑定

```html
<div id="app">
    <!--输入-->
    <input type="text" v-model="input.message">
    <!--把输入的内容展示出来-->
    <p>{{input.message}}</p>
</div>

<script src="../node_modules/vue/dist/vue.min.js"></script>
<script>
    let app = new Vue({
        el:'#app',
        data:{
            input:{
                message:'abc'
            }
        }
    })
</script>
```

效果展示

![image-20200813234546644](http://picture.youyouluming.cn/image-20200813234546644.png)

### 3、事件

模拟一个搜索框，点击搜索显示对应的网址，先准备一个json文件放模拟数据

data.json

```json
{
  "items": [
    {
      "title": "呦呦鹿鸣",
      "site": "http://www.youyouluming.cn"
    },
    {
      "title": "baidu",
      "site": "http://www.baidu.com"
    }
  ]
}
```



```html
<body>
<div id="app">
    <!--输入-->
    搜索：<input type="text" v-model="input.message"><br>
    <!--绑定事件 v-on 或 @-->
    <button v-on:click="searchInfo()"> 搜索</button>
    <!--把输入的内容展示出来-->
    <p>{{input.message}}</p>
    <!--显示结果-->
    <p><a v-bind:href="result.site">{{result.title}}</a></p>
</div>

<script src="../node_modules/vue/dist/vue.min.js"></script>
<script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.5.1/jquery.js"></script>
<script>
    let app = new Vue({
        el: '#app',
        data: {
            input: {
                message: 'abc'
            },
            result: {}
        },
        methods: {//绑定方法
            searchInfo() {
                //取出输入框的值
                let message = this.input.message;
                console.log(message)
                $.get('mock/data.json', data => {
                    console.log(data)
                    data.items.forEach(element => {
                        if (element.title === message) {
                           this.result = element
                        }
                    });
                })
            }
        }
    })
</script>
</body>
```

### 4、修饰符

修饰符是以'.'指明的特殊后缀，用于指出一个指令应该以特殊方式绑定。

阻止事件的默认行为

![image-20200814131435131](http://picture.youyouluming.cn/image-20200814131435131.png)

### 5、条件渲染

v-if：条件渲染

```html
<div id="app">
    <input type="checkbox" v-model="agree">是否确认{{agree}}
    <p v-if="agree">确认</p>
    <p v-else>取消</p>
</div>
<script>
    let app = new Vue({
        el:'#app',
        data:{
            agree:false
        }
    })
</script>
```

v-show：条件指令

```html
<div id="app2">
    <input type="checkbox" v-model="agree">是否确认{{agree}}
    <p v-show="agree">确认</p>
    <p v-show="!agree">取消</p>
</div>
```

v-if和v-show的区别：

* v-if：动态修改了页面的元素，在条件不成立时不加载元素
* v-show：预加载元素，条件不成立时修改元素的属性使其隐藏

### 6、列表渲染

v-for：列表渲染指令

```html
<body>
<div id="app">
    <!--列表渲染-->
    <ul>
        <li v-for="n in nums">{{n}}</li>
    </ul>

    <!--表格渲染-->
    <table>
        <!--user：第一个参数是遍历出来的数据  index：第二个参数是索引号-->
        <tr v-for="(user,index) in userList">
            <td>{{index+1}}</td>
            <td>{{user.id}}</td>
            <td>{{user.name}}</td>
            <td>{{user.age}}</td>
        </tr>
    </table>

    <!--表单渲染-->
    <form action="">
        <p v-for="(value,key,index) in user">
            <label>
                {{index + 1}} - {{key}}: <input type="text" v-model="value">
            </label>
        </p>
    </form>
</div>

<script src="../node_modules/vue/dist/vue.min.js"></script>
<script>
    let app = new Vue({
        el: '#app',
        data: {
            nums: [1, 2, 3, 4, 5, 6],
            userList: [
                {id: 1, name: "jack", age: 11},
                {id: 2, name: "marry", age: 12},
                {id: 3, name: "tom", age: 13}
            ],
            user: {
                id: 4,
                name: "张三",
                age: 14
            }
        }

    })
</script>
</body>
```

### 7、计算属性

使用computed计算属性

```html
<body>
<div id="app">
<p>原文：{{message}}</p>
<p>计算属性反转：{{messageReverse}}</p>
<p>方法反转：{{messageReverse2()}}</p>
</div>

<script src="../node_modules/vue/dist/vue.min.js"></script>
<script>
    let app = new Vue({
        el: '#app',
        data: {
            message:"hello world"
        },
        //计算属性实现反转
        computed: {
            messageReverse(){
                return this.message.split("").reverse().join("");
            }
        },
        //方法实现反转
        methods: {
            messageReverse2(){
                return this.message.split("").reverse().join("");
            }
        }
    })
</script>
</body>
```

计算属性反转和方法反转的区别：

* 计算属性：计算属性会基于缓存，只有相关依赖发生变化时才会重新赋值
* 方法：每一次调用都会再次执行



### 8、侦听属性

当有一些数据是随着其他数据的变动而变动时，可以使用侦听属性。

使用watch监听属性和computed计算属性做一个小案例，监听用户输入并输出显示

```html
<body>
<div id="app">
    <p><input type="text" v-model="firstName"></p>
    <p><input type="text" v-model="lastName"></p>
    <p>{{fullName}}</p>
</div>

<script src="../node_modules/vue/dist/vue.min.js"></script>
<script>
    let app = new Vue({
        el: '#app',
        data: {
            firstName: "hello",
            lastName: "world",
            fullName: "hello + world"
        },
        //计算属性的方式监听
        computed: {
            getFullName() {
                return this.firstName + " · " + this.lastName
            }
        },
        //监听属性
        watch: {
            firstName(val) {
                this.fullName = val + " · " + this.lastName;
            },
            lastName(val) {
                this.fullName = this.firstName + " · " + val;
            }
        }
    })
</script>
</body>
```

watch监听属性和computed计算属性的区别

* watch：一直处于监听状态，比较耗费资源，当需要在数据变化时执行异步或开销较大的操作时，使用监听。
* computed：只有值发生变化才会随之变化，节约资源。

### 9、过滤器

#### 9.1、局部过滤器

使用filters定义过滤器

```html
<body>
<div id="app">
    <table>
        <tr v-for="user in userList">
            <td>{{user.id}}</td>
            <td>{{user.name}}</td>
            <td>{{user.age}}</td>
            <!--使用管道服务把gender传到过滤器-->
            <td>{{user.gender|genderFilter}}</td>
        </tr>
    </table>
</div>

<script src="../node_modules/vue/dist/vue.min.js"></script>
<script>
    let app = new Vue({
        el: '#app',
        data: {
            userList: [
                {id: 1, name: "jack", age: 11, gender: 0},
                {id: 2, name: "marry", age: 12, gender: 1},
                {id: 3, name: "tom", age: 13, gender: 0}
            ]
        },
        //定义过滤器
        filters: {
            genderFilter(gender) {
                return gender === 1 ? '女':'男'
            }
        }
    })
</script>
</body>
```



#### 9.2、全局过滤器

把过滤器提取到一个js中，其他项目都可以使用该过滤器

```js
Vue.filter('caseFilter',value=>{
    return value.charAt(0).toUpperCase()+value.slice(1);
})
```

## 四、Vue进阶

### 1、组件

组件时vue.js最强大功能之一，组件可以扩展html元素，封装可重用代码。组件系统可以让我们使用独立可复用的小组件老构建大型应用，几乎任意类型的应用界面都可以抽象为一个组件树

![image-20200814213108813](http://picture.youyouluming.cn/image-20200814213108813.png)

#### 1.1、局部组件

创建一个菜单组件，可以直接在页面中引用

```html
<body>
<div id="app">
    <Navbar></Navbar>
</div>

<script src="../node_modules/vue/dist/vue.min.js"></script>
<script>
    let app = new Vue({
        el: '#app',
        data: {},
        //创建组件
        components: {
            "Navbar": {
                template: '<ul><li>菜单1</li><li>菜单2</li><li>菜单3</li></ul>'
            }
        }
    })
</script>
</body>
```

#### 1.2、全局组件

把组件提出到一个js，拱其他项目使用

```js
Vue.component("Navbar", {
    template: '<ul><li>菜单1</li><li>菜单2</li><li>菜单3</li></ul>'
})
```

### 2、自定义指令

#### 1.1、局部指令

```html
<body>
<div id="app">
    <input type="text" v-focus>
</div>

<script src="../node_modules/vue/dist/vue.min.js"></script>
<script>
    let app = new Vue({
        el: '#app',
        data: {},
        //定义局部指令
        directives: {
            //定义一个局部自定义指令 v-focus
            focus: {
                //被绑定元素插入到dom中触发
                inserted: function (el) {
                    //聚焦元素
                    el.focus();
                }
            }
        }
    })
</script>
</body>
```

#### 1.2、全局指令

```js
Vue.directive("focus", {
    inserted: function (el) {
        el.focus();
    }
});
```

### 3、Vue的生命周期

每个vue实例在创建时都会经过一系列的初始化过程：创建实例 --> 装载模板 --> 渲染模板等。Vue为生命周期中的每个状态都设置了钩子函数（监听函数），Vue处于不同的生命周期相应的钩子函数就会触发。

生命周期示例图，红色框为钩子方法

<img src="http://picture.youyouluming.cn/Vuelifecycle.png" alt="Vuelifecycle"  />

六个生命周期

```html
<body>
<div id="app">
    <p v-bind:title="message">{{message}}</p>
    <button v-on:click="update">更新</button>
</div>

<script src="../node_modules/vue/dist/vue.min.js"></script>
<script>
    let app = new Vue({
        el: '#app',
        data: {
            message:"床前明月光"
        },
        methods:{
            show() {
                console.log("show")
            },
            update() {
                this.message="玻璃好上霜"
            }
        },
        //===创建时的四个事件
        beforeCreate() { // 第一个被执行的钩子方法：实例被创建出来之前执行
            console.log(this.message) //undefined
            this.show() //TypeError: this.show is not a function
            // beforeCreate执行时，data 和 methods 中的 数据都还没有没初始化
        },
        created() { // 第二个被执行的钩子方法
            console.log(this.message) //床前明月光
            this.show() //执行show方法
            // created执行时，data 和 methods 都已经被初始化好了！
            // 如果要调用 methods 中的方法，或者操作 data 中的数据，最早，只能在 created 中操作
        },
        beforeMount() { // 第三个被执行的钩子方法
            console.log(document.getElementById('h3').innerText) //{{ message }}
            // beforeMount执行时，模板已经在内存中编辑完成了，尚未被渲染到页面中
        },
        mounted() { // 第四个被执行的钩子方法
            console.log(document.getElementById('h3').innerText) //床前明月光
            // 内存中的模板已经渲染到页面，用户已经可以看见内容
        },
        //===运行中的两个事件
        beforeUpdate() { // 数据更新的前一刻
            console.log('界面显示的内容：' + document.getElementById('h3').innerText)
            console.log('data 中的 message 数据是：' + this.message)
            // beforeUpdate执行时，内存中的数据已更新，但是页面尚未被渲染
        },
        updated() {
            console.log('界面显示的内容：' + document.getElementById('h3').innerText)
            console.log('data 中的 message 数据是：' + this.message)
            // updated执行时，内存中的数据已更新，并且页面已经被渲染
        }
    })
</script>
</body>
```

### 4、路由

vue的路由可以让我们通过不同的url访问不同的内容，并且还可以实现多视图的单页web应用

```html
<body>
<div id="app">
       <h1>Hello App!</h1>
       <p>
           <!-- 使用 router-link 组件来导航. -->
           <!-- 通过传入 `to` 属性指定链接. -->
           <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
           <router-link to="/">首页</router-link>
           <router-link to="/student">会员管理</router-link>
           <router-link to="/teacher">讲师管理</router-link>
       </p>
       <!-- 路由出口 -->
       <!-- 路由匹配到的组件将渲染在这里 -->
       <router-view></router-view>
</div>

<script src="../node_modules/vue/dist/vue.min.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
<script>
    // 1. 定义（路由）组件。
    // 可以从其他文件 import 进来
    const Welcome = { template: '<div>欢迎</div>' }
    const Student = { template: '<div>student list</div>' }
    const Teacher = { template: '<div>teacher list</div>' }

    // 2. 定义路由
    // 每个路由应该映射一个组件。
    const routes = [
        { path: '/', redirect: '/welcome' }, //设置默认指向的路径
        { path: '/welcome', component: Welcome },
        { path: '/student', component: Student },
        { path: '/teacher', component: Teacher }
    ]

    // 3. 创建 router 实例，然后传 `routes` 配置
    const router = new VueRouter({
        routes // （缩写）相当于 routes: routes
    })

    // 4. 创建和挂载根实例。
    // 从而让整个应用都有路由功能
    const app = new Vue({
        router
    }).$mount('#app')

</script>
</body>
```

### 5、axios

axios是独立于vue的一个项目，基于promise用于浏览器和node.js的http客户端。

axios的作用：

* 发送异步（Ajax）请求数据。常见方法有get个post。发送时可以指定参数。
* node.js中可以向远程接口发送请求

发送异步请求获取本地数据

```html
<body>
<div id="app">
    <ul>
        <li v-for="(user,index) in users" :key="index">
            {{index}} --- {{user.name}} --- {{user.age}} --- {{user.gender}}
        </li>
    </ul>
</div>

<script src="../node_modules/vue/dist/vue.min.js"></script>
<script src="https://cdn.bootcdn.net/ajax/libs/axios/0.20.0-0/axios.js"></script>
<script>
    let app = new Vue({
        el: '#app',
        data: {
            users: []
        },
        //使用vue的钩子函数去加载数据
        created() {
            //初始化加载数据，then方法表示成功，catch表示失败
            axios({
                url:"mock/user.json",
                method:"get"
            }).then(res => {
                console.log(res);
                //把获取到数据赋值到users，不能使用this，this在axios回调函数表示的是窗口，不是vue的实例
                app.users = res.data.user;
            }).catch(err => {
                alert(err);
            });
        }
    })
</script>
</body>
```

也可以使用异步请求获取服务器上的数据，但是跨域请求需要开启跨域访问权限