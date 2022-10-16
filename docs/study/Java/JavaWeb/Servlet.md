参考 菜鸟教程： https://www.runoob.com/servlet/servlet-environment-setup.html

# Servlet 简介

## Servlet 是什么？

Java Servlet 和CGI类似（Common Gateway Interface，公共网关接口）

相比于 CGI，Servlet 有以下几点优势：

- 性能明显更好。
- Servlet 在 Web 服务器的地址空间内执行。这样它就没有必要再创建一个单独的进程来处理每个客户端请求。
- Servlet 是独立于平台的，因为它们是用 Java 编写的。
- 服务器上的 Java 安全管理器执行了一系列限制，以保护服务器计算机上的资源。因此，Servlet 是可信的。
- Java 类库的全部功能对 Servlet 来说都是可用的。它可以通过 sockets 和 RMI 机制与 applets、数据库或其他软件进行交互

## Servlet 架构

下图显示了 Servlet 在 Web 应用程序中的位置。

![Servlet 架构](Servlet.assets/servlet-arch.jpg)

## Servlet 任务

Servlet 执行以下主要任务：

- 读取客户端（浏览器）发送的显式的数据。这包括网页上的 HTML 表单，或者也可以是来自 applet 或自定义的 HTTP 客户端程序的表单。
- 读取客户端（浏览器）发送的隐式的 HTTP 请求数据。这包括 cookies、媒体类型和浏览器能理解的压缩格式等等。
- 处理数据并生成结果。这个过程可能需要访问数据库，执行 RMI 或 CORBA 调用，调用 Web 服务，或者直接计算得出对应的响应。
- 发送显式的数据（即文档）到客户端（浏览器）。该文档的格式可以是多种多样的，包括文本文件（HTML 或 XML）、二进制文件（GIF 图像）、Excel 等。
- 发送隐式的 HTTP 响应到客户端（浏览器）。这包括告诉浏览器或其他客户端被返回的文档类型（例如 HTML），设置 cookies 和缓存参数，以及其他类似的任务。

## Servlet 包

**Java Servlet 是运行在带有支持 Java Servlet 规范的解释器的 web 服务器上的 Java 类。**

Servlet 可以使用 **javax.servlet** 和 **javax.servlet.http** 包创建，它是 Java 企业版的标准组成部分，Java 企业版是支持大型开发项目的 Java 类库的扩展版本。

这些类实现 Java Servlet 和 JSP 规范。在写本教程的时候，二者相应的版本分别是 Java Servlet 2.5 和 JSP 2.1。

Java Servlet 就像任何其他的 Java 类一样已经被创建和编译。在您安装 Servlet 包并把它们添加到您的计算机上的 Classpath 类路径中之后，您就可以通过 JDK 的 Java 编译器或任何其他编译器来编译 Servlet。



## Servlet／Tomcat/ Spring 之间的关系

参考： https://juejin.cn/post/6844903901133553672

servlet就是一个接口；接口就是规定了一些规范，使得一些具有某些共性的类都能实现这个接口，从而都遵循某些规范。

有的人往往以为就是servlet直接处理客户端的http请求，其实并不是这样，servlet并不会去监听8080端口；直接与客户端打交道是“容器”，比如常用的tomcat。

客户端的请求直接打到tomcat，它监听端口，请求过来后，根据url等信息，确定要将请求交给哪个servlet去处理，然后调用那个servlet的service方法，service方法返回一个response对象，tomcat再把这个response返回给客户端。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/7/27/16c33d3598ad0fca~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)



### Servlet的生命周期


从创建到毁灭：

1. 调用 `init()` 方法初始化
2. 调用 `service()` 方法来处理客户端的请求
3. 调用 `destroy()` 方法释放资源，标记自身为可回收
4. 被垃圾回收器回收


由上面可以看见，servlet的init方法和destroy方法，一般容器调用这两个方法之间的过程，就叫做servlet的生命周期。

调用的整个过程就如上图所示。

当请求来容器第一次调用某个servlet时，需要先初始化init()，

但当某个请求再次打到给servlet时，容器会起多个线程同时访问一个servlet的service（）方法。

 

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/7/27/16c33d3598aef099~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)


由此可以看出，多个客户访问同一service（）方法，会涉及线程安全的问题。


> 如果service()方法没有访问Servlet的成员变量也没有访问全局的资源比如静态变量、文件、数据库连接等，而是只使用了当前线程自己的资源，比如非指向全局资源的临时变量、request和response对象等。该方法本身就是线程安全的，不必进行任何的同步控制。
>
> 如果service()方法访问了Servlet的成员变量，但是对该变量的操作是只读操作，该方法本身就是线程安全的，不必进行任何的同步控制。
>
> 如果service()方法访问了Servlet的成员变量，并且对该变量的操作既有读又有写，通常需要加上同步控制语句。
>
> 如果service()方法访问了全局的静态变量，如果同一时刻系统中也可能有其它线程访问该静态变量，如果既有读也有写的操作，通常需要加上同步控制语句。
>
> 如果service()方法访问了全局的资源，比如文件、数据库连接等，通常需要加上同步控制语句。

 


**面试问题：Servlet如何同时处理多个请求访问？** 

**单实例多线程:** 主要是请求来时，会由线程调度者从线程池李取出来一个线程，来作为响应线程。这个线程可能是已经实例化的，也可能是新创建的。


Servlet容器默认是采用**单实例多线程**的方式处理多个请求的： 
1.当web服务器启动的时候（或客户端发送请求到服务器时），Servlet就被加载并实例化(只存在一个Servlet实例)； 
2.容器初始化化Servlet主要就是读取配置文件（例如tomcat,可以通过servlet.xml的设置线程池中线程数目，初始化线程池通过web.xml,初始化每个参数值等等。 
3.当请求到达时，Servlet容器通过调度线程(Dispatchaer Thread) 调度它管理下线程池中等待执行的线程（Worker Thread）给请求者； 
4.线程执行Servlet的service方法； 
5.请求结束，放回线程池，等待被调用； 
（注意：避免使用实例变量（成员变量），因为如果存在成员变量，可能发生多线程同时访问该资源时，都来操作它，照成数据的不一致，因此产生线程安全问题）

 

从上面可以看出： 
第一：Servlet单实例，减少了产生servlet的开销； 
第二：通过线程池来响应多个请求，提高了请求的响应时间； 
第三：Servlet容器并不关心到达的Servlet请求访问的是否是同一个Servlet还是另一个Servlet，直接分配给它一个新的线程；如果是同一个Servlet的多个请求，那么Servlet的service方法将在多线程中并发的执行； 
第四：每一个请求由ServletRequest对象来接受请求，由ServletResponse对象来响应该请求；



### Spring 

 **任何Spring Web的entry point，都是servlet。**

大名顶顶的spring框架已经风靡多时，一个事物的出现和流行都是会有原因的，那么为什么spring 框架会出现呢？原因就是为了**简化java开发**。

spring的核心就是通过**依赖注入、面向切面编程aop、和模版技术**，解耦业务与系统服务，消除重复代码。借助aop，可以将遍布应用的关注点（如事物和安全）从它们的应用对象中解耦出来。



**Spring 中的Bean**

 1) POJO和JavaBean的区别 : 

"Plain Ordinary Java Object"，简单普通的java对象。主要用来指代那些没有遵循特定的java对象模型，约定或者框架的对象。

POJO的内在含义是指那些:
有一些private的参数作为对象的属性，然后针对每一个参数定义get和set方法访问的接口。
没有从任何类继承、也没有实现任何接口，更没有被其它框架侵入的java对象。

JavaBean 是一种JAVA语言写成的可重用组件。JavaBean符合一定规范编写的Java类，不是一种技术，而是一种规范。大家针对这种规范，总结了很多开发技巧、工具函数。符合这种规范的类，可以被其它的程序员或者框架使用。它的方法命名，构造及行为必须符合特定的约定：

1. 所有属性为private。
2. 这个类必须有一个公共的缺省构造函数。即是提供无参数的构造器。
3. 这个类的属性使用getter和setter来访问，其他方法遵从标准命名规范。
4. 这个类应是可序列化的。实现serializable接口。

因为这些要求主要是靠约定而不是靠实现接口，所以许多开发者把JavaBean看作遵从特定命名约定的POJO。

 

spring中，应用对西那个生存于spring容器中,spring 容器创建对象，装配它们，管理它们的整个生命周期。spring容器通过依赖注入，管理构成应用的组件，它会创建相互协作的组件之间的关联。

2) Bean的生命周期

 

 

 **Spring MVC**

 

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/7/27/16c33d3598c406d2~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

 

 

 

 Spring MVC的运行流程：

 

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/7/27/16c33d3599960804~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

 

 

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/7/27/16c33d3599a4ea1f~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)





# Servlet 环境设置

配置jdk

配置tomcat 



# Servlet 生命周期

Servlet 生命周期可被定义为从创建直到毁灭的整个过程。以下是 Servlet 遵循的过程：

- Servlet 初始化后调用 **init ()** 方法。
- Servlet 调用 **service()** 方法来处理客户端的请求。
- Servlet 销毁前调用 **destroy()** 方法。
- 最后，Servlet 是由 JVM 的垃圾回收器进行垃圾回收的。

现在让我们详细讨论生命周期的方法。



## init() 方法

init 方法被设计成只调用一次。它在第一次创建 Servlet 时被调用，在后续每次用户请求时不再调用。因此，它是用于一次性初始化，就像 Applet 的 init 方法一样。

Servlet 创建于用户第一次调用对应于该 Servlet 的 URL 时，但是您也可以指定 Servlet 在服务器第一次启动时被加载。

当用户调用一个 Servlet 时，就会创建一个 Servlet 实例，每一个用户请求都会产生一个新的线程，适当的时候移交给 doGet 或 doPost 方法。init() 方法简单地创建或加载一些数据，这些数据将被用于 Servlet 的整个生命周期。

init 方法的定义如下：

```
public void init() throws ServletException {
  // 初始化代码...
}
```



## service() 方法

service() 方法是执行实际任务的主要方法。Servlet 容器（即 Web 服务器）调用 service() 方法来处理来自客户端（浏览器）的请求，并把格式化的响应写回给客户端。

每次服务器接收到一个 Servlet 请求时，服务器会产生一个新的线程并调用服务。service() 方法检查 HTTP 请求类型（GET、POST、PUT、DELETE 等），并在适当的时候调用 doGet、doPost、doPut，doDelete 等方法。

下面是该方法的特征：

```
public void service(ServletRequest request, 
                    ServletResponse response) 
      throws ServletException, IOException{
}
```

service() 方法由容器调用，service 方法在适当的时候调用 doGet、doPost、doPut、doDelete 等方法。所以，您不用对 service() 方法做任何动作，您只需要根据来自客户端的请求类型来重写 doGet() 或 doPost() 即可。

doGet() 和 doPost() 方法是每次服务请求中最常用的方法。下面是这两种方法的特征。



## doGet() 方法

GET 请求来自于一个 URL 的正常请求，或者来自于一个未指定 METHOD 的 HTML 表单，它由 doGet() 方法处理。

```
public void doGet(HttpServletRequest request,
                  HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}
```

## doPost() 方法

POST 请求来自于一个特别指定了 METHOD 为 POST 的 HTML 表单，它由 doPost() 方法处理。

```
public void doPost(HttpServletRequest request,
                   HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}
```

## destroy() 方法

destroy() 方法只会被调用一次，在 Servlet 生命周期结束时被调用。destroy() 方法可以让您的 Servlet 关闭数据库连接、停止后台线程、把 Cookie 列表或点击计数器写入到磁盘，并执行其他类似的清理活动。

在调用 destroy() 方法之后，servlet 对象被标记为垃圾回收。destroy 方法定义如下所示：

```
  public void destroy() {
    // 终止化代码...
  }
```



## 架构图

下图显示了一个典型的 Servlet 生命周期方案。

- 第一个到达服务器的 HTTP 请求被委派到 Servlet 容器。
- Servlet 容器在调用 service() 方法之前加载 Servlet。
- 然后 Servlet 容器处理由多个线程产生的多个请求，每个线程执行一个单一的 Servlet 实例的 service() 方法。

![Servlet 生命周期](Servlet.assets/Servlet-LifeCycle.jpg)



# Servlet 实例

Servlet 是服务 HTTP 请求并实现 **javax.servlet.Servlet** 接口的 Java 类。Web 应用程序开发人员通常编写 Servlet 来扩展 javax.servlet.http.HttpServlet，并实现 Servlet 接口的抽象类专门用来处理 HTTP 请求。





# 未完待续。。。















