# Servlet
## 一、servlet简介

Servlet是sun公司提供的一门用于开发动态web资源的技术。　　
Sun公司在其API中提供了一个servlet接口，用户若想用发一个动态web资源(即开发一个Java程序向浏览器输出数据)，需要完成以下2个步骤：
1. 编写一个Java类，实现servlet接口。　　
2. 把开发好的Java类部署到web服务器中。　　

## 二、servlet执行过程
客户端在浏览器中向服务器发送servlet请求，web收到客户端的servlet的访问请求后：
![a199ea0db56af73da94e1d2547856f4e.png](http://picture.youyouluming.cn/afawfawfwa312515v.png)

## 三、使用servlet
### 3.1、在idea中创建一个maven工程，并添加依赖
servlet和jsp的依赖
```xml
 <dependencies>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.0</version>
            <scope>provided</scope>
        </dependency>
```

### 3.2、servlet接口实现类
servlet提供了两个实现类GenericServlet、HttpServlet。
在使用中通常继承HttpServlet，并且重写该类下的两个方法doGet或doPost
```java
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("doget");
        //获取流对象将数据打印在浏览器
        PrintWriter writer = resp.getWriter();
        writer.print("abc");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}
```

### 3.3、servlet的映射配置
由于客户端访问的是web上的资源，所以servlet想要被外界访问，必须将程序映射到一个url地址上，在web.xml中完成该工作。使用<servlet>和<serlvet-mapping>标签。
<servlet>元素用于注册Servlet，它包含有两个主要的子元素：<servlet-name>和<servlet-class>，分别用于设置Servlet的注册名称和Servlet的完整类名。
<servlet-mapping>元素用于映射一个已注册的Servlet的一个对外访问路径，它包含有两个子元素：<servlet-name>和<url-pattern>，分别用于指定Servlet的注册名称和Servlet的对外访问路径。
```xml
 <!-- 注册servlet -->
    <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>cn.luming.servlet.HelloServlet</servlet-class>
    </servlet>
    <!-- servlet请求路径 -->
    <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
    </servlet-mapping>
```
### 3.4、servlet使用细节
1. 可以使用 * 通配符映射
    1. *.扩展名：匹配以指定扩展名结尾的资源
    2. 正斜杠（/）开头并以"/*"结尾：匹配该目录下所有的资源
2. 缺省的servlet：如果某个映射路径仅为一个/，那么这个servlet就是当前web应用的缺省servlet。
凡是在xml文件中找不到配置映射元素的url时，都会交给缺省servlet来处理，缺省servlet用来处理其他servlet都不处理的访问请求。

## 四、ServletContext
web容器在启动时，会给每个web应用程序创建一个ServletContext对象，代表当前web应用。同一个web共享一个ServletContext对象，所有可以使用ServletContext对象来实现通讯。

### 4.1、多个servlet通过ServletContext对象实现数据共享
存储数据的servlet
```java
public class ServletContext01 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext();
        String username = "张三";
        servletContext.setAttribute("username", username);
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

获取的servlet
```java
public class GetContext extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext();
        //设置响应编码
        resp.setContentType("text/html");
        resp.setCharacterEncoding("utf-8");
        //获取数据
        String username = (String) servletContext.getAttribute("username");
        PrintWriter writer = resp.getWriter();
        writer.print("username" + ":" + username);
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

web.xml需要把两个servlet的映射都配置上
```xml
<!-- 注册servlet -->
  <servlet>
    <servlet-name>context01</servlet-name>
    <servlet-class>cn.luming.servlet.ServletContext01</servlet-class>
  </servlet>
  <!-- servlet请求路径 -->
  <servlet-mapping>
    <servlet-name>context01</servlet-name>
    <url-pattern>/context01</url-pattern>
  </servlet-mapping>
  <!-- 注册servlet -->
  <servlet>
    <servlet-name>getContext</servlet-name>
    <servlet-class>cn.luming.servlet.GetContext</servlet-class>
  </servlet>
  <!-- servlet请求路径 -->
  <servlet-mapping>
    <servlet-name>getContext</servlet-name>
    <url-pattern>/getContext</url-pattern>
  </servlet-mapping>

```

### 4.2、获取web应用的初始化参数

在web.xml文件中使用<context-param>标签配置WEB应用的初始化参数
```xml
 <context-param>
    <param-name>username</param-name>
    <param-value>abc</param-value>
  </context-param>
```
获取web应用初始化参数
```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext();
        String username = servletContext.getInitParameter("username");
        resp.getWriter().print(username);
    }
```

### 4.3、使用ServletContext实现请求转发
实现转发的servlet
```java
 protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext servletContext = this.getServletContext();
        //获取请求转发对象getRequestDispatcher，请求转发forward
        servletContext.getRequestDispatcher("/printServlet").forward(req, resp);
    }
```
被转发的servlet
```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().print("PrintServlet");
    }
```
注意：此处转发的路径写的是xml配置里映射的路径。访问的是实现转发的servlet，浏览器却是被转发的对象，而重定向的浏览器路径会跳转到被转发的servlet。

### 4.4、读取资源文件
```java
 protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        InputStream is = this.getServletContext().getResourceAsStream("/WEB-INF/classes/jdbc.properties");
        Properties prop = new Properties();
        prop.load(is);
        String username = prop.getProperty("username");
        resp.getWriter().print(username);
    }
```

## 五、HttpServletResponse
### 1、HttpServletResponse对象介绍

Web服务器收到客户端的http请求，会针对每一次请求，分别创建一个用于代表请求的request对象、和代表响应的response对象。获取客户端请求的过来的数据，就使用request。如果向客户端输出数据，就使用response。
HttpServletResponse对象代表服务器的响应。这个对象中封装了向客户端发送数据、发送响应头，发送响应状态码的方法。

#### 1.1、向客户端发送数据
```java
ServletOutputStream getOutputStream() throws IOException;
PrintWriter getWriter() throws IOException;
```
#### 1.2、负责向客户端发送响应头
```java
void setCharacterEncoding(String var1);
void setContentLength(int var1);
void setContentLengthLong(long var1);
void setContentType(String var1);
void setHeader(String var1, String var2);
void addHeader(String var1, String var2);
void setIntHeader(String var1, int var2);
```

#### 1.3、负责向客户端发送响应状态码
```java
void setStatus(int var1);
```
#### 1.4、响应状态码的常量
状态码404对应的常量
```java
int SC_NOT_FOUND = 404;
```
状态码200对应的常量
```java
int SC_OK = 200;
```
状态码500对应的常量
```java
int SC_INTERNAL_SERVER_ERROR = 500;
```
### 2、HttpServletResponse对象常见应用
#### 2.1、向客户端输出消息
#### 2.2、下载文件
1. 获取下载文件的路径
2. 获取下载文件名称
3. 设置content-disposition响应头控制浏览器以下载的形式打开文件
4. 获取下载的文件输入流
5. 创建缓冲区
6. 获取OutPutStream流
7. 将FileOutPutStream流写入buffer缓冲区
8. 使用OutputStream将缓冲区的数据输出到客户端浏览器

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 1. 获取下载文件的路径
        String realPath = "D:\\JavaCode\\web\\web01_Servlet\\response\\src\\main\\resources\\啊啊.png";
        System.out.println("下载路径" + realPath);
        // 2. 获取下载文件名称
        String fileName = realPath.substring(realPath.lastIndexOf("\\") + 1);
        // 3. 设置content-disposition响应头控制浏览器以下载的形式打开文件，并且设置编码
        response.setHeader("content-disposition", "attachment;filename=" + URLEncoder.encode(fileName, "utf-8"));
        // 4. 获取下载的文件输入流
        FileInputStream in = new FileInputStream(realPath);
        // 5. 创建缓冲区
        int len = 0;
        byte[] buffer = new byte[1024];
        // 6. 获取OutPutStream流
        ServletOutputStream out = response.getOutputStream();
        // 7. 将FileOutPutStream流写入buffer缓冲区
        // 8. 使用OutputStream将缓冲区的数据输出到客户端浏览器
        while ((len = in.read(buffer)) > 0) {
            out.write(buffer, 0, len);
        }
        in.close();
        out.close();

    }
```

#### 2.3、生成验证码
```java
 protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //浏览器3秒刷新一次
        response.setHeader("refresh", "3");
        //在内存中创建图片
        BufferedImage image = new BufferedImage(80, 20, BufferedImage.TYPE_INT_RGB);
        //得到图片
        Graphics2D graphics = (Graphics2D) image.getGraphics();//笔
        //设置图片的背景颜色
        graphics.setColor(Color.white);
        graphics.fillRect(0, 0, 80, 20);
        //在图片上写数据
        graphics.setColor(Color.BLUE);
        graphics.setFont(new Font(null, Font.BOLD, 20));
        graphics.drawString(makeNum(), 0, 20);
        //设置响应头控制浏览器以图片的方式打开
        response.setContentType("image/jpg");
        //不让浏览器缓存
        response.setDateHeader("expires", -1);
        response.setHeader("Cache-Control", "no-cache");
        response.setHeader("Pragma", "no-cache");
        //把图片写给浏览器
        ImageIO.write(image, "jpg", response.getOutputStream());
    }

    /**
     * 生成随机数
     *
     * @return
     */
    private String makeNum() {
        Random random = new Random();
        String num = random.nextInt(9999999) + "";
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < 7 - num.length(); i++) {
            sb.append("0");
        }
        num = sb.toString() + num;
        return num;
    }
```

### 3、设置响应头控制浏览器的行为
### 3.1、设置http响应头控制浏览器禁止缓存当前文档内容
```java
response.setDateHeader("expires", -1);
response.setHeader("Cache-Control", "no-cache");
response.setHeader("Pragma", "no-cache");
```
#### 3.2、设置http响应头控制浏览器定时刷新网页
```java
response.setHeader("refresh", "3");
```
#### 3.3、通过response实现请求重定向
1. 重定向是指：一个web资源收到客户端请求后，通知客户端去访问另一个web资源。
2. 实现方式：response.sendRedirect(String location)，即调用response对象的sendRedirect方法实现请求重定向　　
3. sendRedirect内部的实现原理：使用response设置302状态码和设置location响应头实现重定向

```java
 protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        /*
            response.setHeader("Location","/img");
            response.setStatus(302);
        */
        response.sendRedirect("/ImageServlet");
    }
```
使用提交表单重定向到另一个jsp页面并获取表单内容
```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("RequestServlet");
        //处理请求
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        System.out.println(username + ":" + password);
        response.sendRedirect("success.jsp");
    }
```

## 六、HttpServletRequest
### 1、HttpServletRequest介绍
HttpServletRequest对象代表客户端的请求，当客户端通过HTTP协议访问服务器时，HTTP请求头中的所有信息都封装在这个对象中，通过这个对象提供的方法，可以获得客户端请求的所有信息。

### 2、HttpServletRequest介绍常用方法
#### 2.1、获取客户端请求参数（客户提交的数据）
* getParameter(String)方法(常用)
* getParameterValues(String name)方法(常用)
* getParameterMap()方法

从这个表单中获取数据并返回控制台
```html
<div style="text-align: center">
    <form action="${pageContext.request.contextPath}/loginServlet" method="post">
        username: <input type="text" name="username"><br>
        password: <input type="password" name="password"><br>
        hobby:
        <input type="checkbox" name="hobby" value="c">c
        <input type="checkbox" name="hobby" value="t">t
        <input type="checkbox" name="hobby" value="r">t
        <input type="checkbox" name="hobby" value="b">b
        <input type="submit" value="submit">
    </form>
</div>
```

```java
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //解决中文乱码
        request.setCharacterEncoding("utf-8");
        response.setCharacterEncoding("utf-8");
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        String[] hobbies = request.getParameterValues("hobby");
        System.out.println(username + "---" + password + "---" + Arrays.toString(hobbies));
        //请求转发
        request.getRequestDispatcher(request.getContextPath() + "success.jsp").forward(request, response);
    }
```

## 七、Cookie
### 1、会话
会话就是当用户使用一个浏览器访问多个web资源，并关闭浏览器，称之为一个会话。
在进行会话中会产生一些数据，程序会想法把数据保存下来。比如，网站登录状态在关闭浏览器后再次打开还存在。

### 2. 保存会话数据的两个技术
#### 2.1 Cookie
Cookie是客户端技术，程序把每个用户的数据以cookie的形式写给用户各自的浏览器。当用户使用浏览器再去访问服务器中的web资源时，就会带着各自的数据去。这样，web资源处理的就是用户各自的数据了。
#### 2.2 Session
Session是服务器端技术，利用这个技术，服务器在运行时可以为每一个用户的浏览器创建一个其独享的session对象，由于session为用户浏览器独享，所以用户在访问服务器的web资源时，可以把各自的数据放在各自的session中，当用户再去访问服务器中的其它web资源时，其它web资源再从用户各自的session中取出数据为用户服务。

### 3、Cookie的使用范例（记录上一次访问的时间）
```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //设置服务器端以UTF-8编码进行输出
        response.setCharacterEncoding("UTF-8");
        //设置浏览器以UTF-8编码进行接收,解决中文乱码问题
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        Cookie[] cookies = request.getCookies();
        if (cookies != null) {
            //如果访问过就记录上次访问的时间
            out.write("上次访问的时间是:");
            for (int i = 0; i < cookies.length; i++) {
                Cookie cookie = cookies[i];
                if (cookie.getName().equals("lastAccessTime")) {
                    //转为date对象
                    long lastAccessTime = Long.parseLong(cookie.getValue());
                    Date date = new Date(lastAccessTime);
                    out.write(date.toLocaleString());
                }
            }
        } else {
            out.write("首次访问");
        }
        //用户访问过后重新设置访问时间，并存储到cookie
        Cookie cookie = new Cookie("lastAccessTime", System.currentTimeMillis() + "");
        //将cookie对象添加到response对象中，这样服务器在输出response对象中的内容时就会把cookie也输出到客户端浏览器
        response.addCookie(cookie);
    }
```
没有使用setMaxAge设置有效期，浏览器一关闭cookie就失效了，要向在浏览器关闭后cookie还生效，就要使用setMaxAge设置一个有效期
```java
//用户访问过后重新设置访问时间，并存储到cookie
Cookie cookie = new Cookie("lastAccessTime", System.currentTimeMillis() + "");
//设置cookie的有效期,一天
cookie.setMaxAge(24 * 60 * 60);
//将cookie对象添加到response对象中，这样服务器在输出response对象中的内容时就会把cookie也输出到客户端浏览器
response.addCookie(cookie);
```

### 4、Cookie注意细节
1. 一个Cookie只能标识一种信息，至少包含一个该信息的名字和值
2. 一个web站点可以给浏览器发送多个cookie。
3. 浏览器最多存储200个cookie，每个站点最多存20个Cookie，每个cookie大小限制为4kb
4. 如果创建了一个cookie，并将他发送到浏览器，默认情况下它是一个会话级别的cookie（即存储在浏览器的内存中），用户退出浏览器之后即被删除。若希望浏览器将该cookie存储在磁盘上，则需要使用maxAge，并给出一个以秒为单位的时间。将最大时效设为0则是命令浏览器删除该cookie。

### 5、删除cookie
创建一个Cookie，名字必须要和被删除的Cookie一致
```java
 protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //创建一个Cookie，必须和被删除的Cookie名字一样
        Cookie cookie = new Cookie("lastAccessTime", System.currentTimeMillis() + "");
        //将Cookie的有效期设置为0，立马过期
        cookie.setMaxAge(0);
        response.addCookie(cookie);
    }
```

## 八、Session
### 1、Session简介
服务器会给每个浏览器创建一个会话对象（Session对象），默认情况下，一个浏览器独占一个Session对象。在需要保存用户数据时，服务器程序可以把用户数据写道用户浏览器独占的Session中，当用户再访问该网站其他站点是吗，都可使用Session中的数据。

### 2、Session和Cookie的区别
* Cookie是把用户的数据写给用户浏览器的
* Session是把用户的数据写到用户独占的Session中
* Session对象是由服务器创建，开发人员可以获取Session对象

### 3、Session的实现
#### 3.1、服务器是如何实现一个session为一个用户浏览器服务的？
服务器创建session出来后，会把session的id号，以cookie的形式回写给客户机，这样，只要客户机的浏览器不关，再去访问服务器时，都会带着session的id号去，服务器发现客户机浏览器带session id过来了，就会使用内存中与之对应的session为之服务。
```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //设置服务器端以UTF-8编码进行输出
        response.setCharacterEncoding("UTF-8");
        //设置浏览器以UTF-8编码进行接收,解决中文乱码问题
        response.setContentType("text/html;charset=UTF-8");
        //获取session
        HttpSession session = request.getSession();
        //将数据存入session
        session.setAttribute("username", "张三");
        //获取session的id
        String id = session.getId();
        //判断session是不是新的
        if (session.isNew()) {
            //新的就自动创建一个
            response.getWriter().print("session创建，id为" + id);
        } else {
            response.getWriter().print("session已经存在，id为" + id);
        }
    }
```

第一次访问时，服务器会创建一个新的sesion，并且把session的Id以cookie的形式发送给客户端浏览器
再次访问时，发现把存储到Cookie中的Session的id一起传递到服务端。
![3caf23431a1ebb093ae40d39c427366c.png](http://picture.youyouluming.cn/adafwaf415161.png)
![79d6026858fcad51868af23848550cc5.png](http://picture.youyouluming.cn/afawqrq14152adaw.png)

### 3.2、Session的销毁
Session的默认销毁时间时30分钟没有使用，服务器自动销毁session。在web.xml上可以手动配置session的失效时间。
```xml
<session-config>
        <!-- 设置15分钟销毁 -->
        <session-timeout>15</session-timeout>
    </session-config>
```
也可以在程序中手动销毁
```java
HttpSession session = request.getSession();
session.invalidate();
```