## servlet

HttpServlet容器响应Web客户请求流程如下：

1. Web客户向Servlet容器发出Http请求；
2. Servlet容器解析Web客户的Http请求；
3. Servlet容器创建一个HttpRequest对象，在这个对象中封装Http请求信息；
4. Servlet容器创建一个HttpResponse对象；
5. Servlet容器调用HttpServlet的service方法，这个方法中会根据request的Method来判断具体是执行doGet还是doPost，把HttpRequest和HttpResponse对象作为service方法的参数传给HttpServlet对象；
6. HttpServlet调用HttpRequest的有关方法，获取HTTP请求信息；
7. HttpServlet调用HttpResponse的有关方法，生成响应数据；
8. Servlet容器把HttpServlet的响应结果传给Web客户。

doGet() 或 doPost() 是创建HttpServlet时需要覆盖的方法.

Servlet在web程序中的位置：

![](https://raw.githubusercontent.com/callme001/my-images/master/imgs/2016-09-0414%3A48%3A33.jpeg))

### 1. Servlet生命周期

Servlet 生命周期可被定义为从它被创建直到被销毁的整个过程。以下是 servlet 遵循的过程:
* 通过调用 init () 方法 servlet 被初始化。
* Servlet 调用 service() 方法来处理客户端的请求。
* 通过调用 destroy() 方法 servlet 终止。
* 最后,servlet 是由 JVM 的垃圾回收器进行垃圾回收的。

#### 1.1 init() 方法

init 方法被设计成**只调用一次**。它在第一次创建 servlet 时被调用,在后续每次用户请求时不再调用。因此,它用于一次性初始化,与 applets 的 init 方法一样。

通常情况下,当用户第一次调用对应于该 servlet 的 URL 时,servlet 被创建,但是当服务器第一次启动时,你也可以指定 servlet 被加载。

当用户调用 servlet 时,每个 servlet 的一个实例就会被创建,并且每一个用户请求都会产生一个新的线程,该线程在适当的时候移交给 doGet 或 doPost 方法。init() 方法简单地创建或加载一些数据,这些数据将被用于 servlet 的整个生命周期。

init 方法的定义如下:
```java
public void init() throws ServletException {
	// Initialization code...
}
```

#### 1.2 service() 方法

service() 方法是执行实际任务的主要方法。Servlet 容器(即 web 服务器)调用 service() 方法来处理来自客户端(浏览器)的请求,并将格式化的响应写回到客户端。

每次服务器接收到一个 servlet 请求时,服务器会产生一个新的线程并调用服务。service() 方法检查 HTTP 请求类型(GET、POST、PUT、DELETE 等),并在适当的时候调用 doGet、doPost、doPut、doDelete 等方法。
下面是该方法的特征:
```java
public void service(ServletRequest request,ServletResponse response)throws ServletException, IOException{
	...
}
```
service() 方法由容器调用,且 service 方法在适当的时候调用 doGet、doPost、doPut、doDelete 等。所以对 service() 方法你什么都不需要做,只是根据你接收到的来自客户端的请求类型来重写 doGet() 或 doPost()。

#### 1.3 doGet()和doPost()方法

doGet() 和 doPost() 方法在每次服务请求中是最常用的方法。下面是这两种方法的特征。doGet() 方法GET 请求来自于一个 URL 的正常请求,或者来自于一个没有 METHOD 指定的 HTML 表单,且它由 doGet()方法处理。

```java
public void doGet(HttpServletRequest request,HttpServletResponse response)throws ServletException, IOException {
	// Servlet code
}
```
doPost() 方法
POST 请求来自于一个 HTML 表单,该表单特别的将 POST 列为 METHOD 且它由 doPost() 方法处理。

```java
public void doPost(HttpServletRequest request,HttpServletResponse response)throws ServletException, IOException {
	// Servlet code
}
```

#### 1.4 destroy() 方法
destroy() 方法只在 servlet 生命周期结束时被调用一次。destroy() 方法可以让你的 servlet 关闭数据库连接、停止后台线程、将 cookie 列表或点击计数器写入磁盘,并执行其他类似的清理活动。
在调用 destroy() 方法之后,servlet 对象被标记用于垃圾回收。destroy 方法的定义如下所示:
```java
public void destroy() {
	//
}
```

#### 1.5 结构框图

下图显示了一个典型的 servlet 生命周期场景。
* 最先到达服务器的 HTTP 请求被委派到 servlet 容器。
* 在调用 service() 方法之前 servlet 容器加载 servlet。
* 然后 servlet 容器通过产生多个线程来处理多个请求,每个线程执行 servlet 的单个实例的 service() 方法。

![](https://raw.githubusercontent.com/callme001/my-images/master/imgs/2016-09-04-15%3A15%3A40.jpeg)


### 2. 获取用户表单输入

在doGet()和doPost()中，提供了四个函数来获取用户的输入：
```java
request.getParameter(arg0); // 输入一个表单name获取表单值
request.getParameterValues(arg0); //输入的可能是复选项，获取同一name的多个表单值
request.getParameterMap(); //获取所有表单项 返回的数据是Map<String,String[]> 返回表单name和表单值数组的map集合
request.getParameterNames(); //获取所有的表单name 返回一个 String 对象的枚举,包含在该请求中包含的参数的名称。
```
需要注意的是：客户端在使用post表单提交数据的时候，需要指定表单`Content-Type: application/x-www-form-urlencoded`。如果不指定，servlet后台拿不到数据

### 3. 读取 HTTP 头信息

部分函数：

* **Cookie[] getCookies()** : 返回一个数组,包含客户端发送该请求的所有的 Cookie 对象
* Enumeration getAttributeNames() : 返回一个枚举,包含提供给该请求可用的属性名称
* **Enumeration getHeaderNames()** : 返回一个枚举,包含在该请求中包含的所有的头名
* Enumeration getParameterNames() : 返回一个 String 对象的枚举,包含在该请求中包含的参数的名称。
* **HttpSession getSession()** : 返回与该请求关联的当前 session 会话,或者如果请求没有 session 会话,则创建一个
* Locale getLocale() ： 基于 Accept-Language 头,返回客户端接受内容的首选的区域设置
* **String getCharacterEncoding()** ： 返回请求主体中使用的字符编码的名称
* String getContentType() ： 返回请求主体的 MIME 类型,如果不知道类型则返回 null
* String getContextPath() ： 返回指示请求上下文的请求 URI 部分
* String getHeader(String name) ： 以字符串形式返回指定的请求头的值
* String getMethod() ： 返回请求的 HTTP 方法的名称,例如,GET、POST 或 PUT
* String getQueryString() ： 返回包含在路径后的请求 URL 中的查询字符串
* String getRemoteAddr() ： 返回发送请求的客户端的互联网协议(IP)地址
* int getServerPort() ： 返回接收到这个请求的端口号


### 4. 设置响应头

![](https://raw.githubusercontent.com/callme001/my-images/master/imgs/2016-09-04%2016-00-48%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)
![](https://raw.githubusercontent.com/callme001/my-images/master/imgs/2016-09-04%2016-01-10%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

部分函数（都是在HttpServletResponse实例）：

* boolean containsHeader(String name) : 返回一个布尔值,指示是否已经设置已命名的响应头信息
* boolean isCommitted() : 返回一个布尔值,指示响应是否已经提交
* **void addCookie(Cookie cookie)** : 把指定的 cookie 添加到响应
* void addDateHeader(String name, long date) : 添加一个带有给定的名称和日期值的响应头信息
* void addHeader(String name, String value) : 加一个带有给定的名称和值的响应头信息
* void addIntHeader(String name, int value) : 添加一个带有给定的名称和整数值的响应头信息
* void flushBuffer() : 强制任何在缓冲区中的内容被写入到客户端
* void reset() : 清除缓冲区中存在的任何数据,包括状态码和头信息
* void resetBuffer() : 清除响应中基础缓冲区的内容,不清除状态码和头信息
* void sendError(int sc) : 使用指定的状态码发送错误响应到客户端,并清除缓冲区
* void sendError(int sc, String msg) : 使用指定的状态发送错误响应到客户端
* **void setCharacterEncoding(String charset)** : 设置被发送到客户端的响应的字符编码(MIME 字符集)例如,UTF-8
* **void setContentType(String type)** : 如果响应还未被提交,设置被发送到客户端的响应的内容类型
* **void setStatus(int sc)** : 为该响应设置状态码

### 5. 过滤器

一个基础的过滤器如下：

```java
package helloServlet;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;

public class myFilter implements Filter{

	@Override
	public void destroy() {
		// TODO Auto-generated method stub
	}

	@Override
	public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2)
			throws IOException, ServletException {
		// TODO Auto-generated method stub

		PrintWriter out = arg1.getWriter();
		out.println("filter has work");

		arg2.doFilter(arg0,arg1);
	}

	@Override
	public void init(FilterConfig arg0) throws ServletException {
		// TODO Auto-generated method stub
	}
}

```
在doFilter()方法中实现过滤逻辑，init()和destory()方法都是由Web容器调用。

还需要在web.xml中配置过滤器映射(/* 表示拦截所有请求)

```xml
  <filter>
  	<filter-name>filter</filter-name>
  	<filter-class>helloServlet.myFilter</filter-class>
  </filter>
  <filter-mapping>
  	<filter-name>filter</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
```

对于多个过滤器，需要注意的是：web.xml 中的 filter-mapping 元素的顺序决定了 web 容器把过滤器应用到 servlet 的顺序。

编写一个简单的servlet代码如下：

```java
package helloServlet;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class helloServlet extends HttpServlet {

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

		PrintWriter out = resp.getWriter();
		out.println("this is get method");

	}


	@Override
	public void service(ServletRequest arg0, ServletResponse arg1) throws ServletException, IOException {
		PrintWriter out = arg1.getWriter();
		out.println("service method has work");
		super.service(arg0, arg1);
	}

}

```
在web.xml配置好servlet映射，访问的时候可得到以下输出：
![](https://raw.githubusercontent.com/callme001/my-images/master/imgs/2016-09-04%2016-51-19%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png)

说明filter在service方法之前执行了，其次是service()方法，最后才是doGet()方法

## 题目总结

JSP内置对象：

* request对象 : 客户端的请求信息被封装在request对象中，通过它才能了解到客户的需求，然后做出响应。它是HttpServletRequest类的实例。
* response对象 : response对象包含了响应客户请求的有关信息，但在JSP中很少直接用到它。它是HttpServletResponse类的实例。
* session对象 : session对象指的是客户端与服务器的一次会话，从客户连到服务器的一个WebApplication开始，直到客户端与服务器断开连接为止。它是HttpSession类的实例.
* out对象 : out对象是JspWriter类的实例,是向客户端输出内容常用的对象
* page对象 : page对象就是指向当前JSP页面本身，有点象类中的this指针，它是java.lang.Object类的实例
* application对象 : application对象实现了用户间数据的共享，可存放全局变量。它开始于服务器的启动，直到服务器的关闭，在此期间，此对象将一直存在；这样在用户的前后连接或不同用户之间的连接中，可以对此对象的同一属性进行操作；在任何地方对此对象属性的操作，都将影响到其他用户对此的访问。服务器的启动和关闭决定了application对象的生命。它是ServletContext类的实例。
* exception对象 : exception对象是一个例外对象，当一个页面在运行过程中发生了例外，就产生这个对象。如果一个JSP页面要应用此对象，就必须把isErrorPage设为true，否则无法编译。他实际上是java.lang.Throwable的对象
* pageContext对象 : pageContext对象提供了对JSP页面内所有的对象及名字空间的访问，也就是说他可以访问到本页所在的SESSION，也可以取本页面所在的application的某一属性值，他相当于页面中所有功能的集大成者，它的本 类名也叫pageContext。
* config对象 : config对象是在一个Servlet初始化时，JSP引擎向它传递信息用的，此信息包括Servlet初始化时所要用到的参数（通过属性名和属性值构成）以及服务器的有关信息（通过传递一个ServletContext对象）