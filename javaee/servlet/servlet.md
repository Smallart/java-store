# [-[Servlet]-]

在刚开始的时候，[-[Web]-]页面还都是**静态页面**-服务器不对网页文件做任何处理，读取文件之后就直接当作响应传递给浏览器，也就是网页的内容在上传之前就写死了，每个人访问的都是同一内容。但是随着需求的改变，网站开发者就需要一种能够动态展示的网页-服务器在响应之前，先依靠客户端的请求参数、表头或实际服务器上的状态，以程序的方式动态产生响应内容的技术，所以各种能够在服务端动态处理请求的技术就孕育而生，其中[-[Servlet]-]就是这样的一种技术。

![动态页面-静态页面](/img/Dynamic-vs-Static-webpage.png)

总结一下：

> [-[Servlet]-]是[-[Java]-]的编程语言类，是一种用于扩展承担各种[-[request-reponse]-]程序模式的服务器应用能力的技术。

## 什么是[-[Web]-]容器

尽管[-[Servlet]-]可以响应任何种类的请求，但是我们最熟悉的还是利用[-[HTTP]-]协议的[-[Web]-]服务器，[-[Servlet]-]对应实现的API在[-[Java]-]的`java.servlet`、`java.servlet.http`包中。由于[-[Servlet]-]只是[-[Java]-]编程语言类，那么就需要一个[-[Main]-]入口去实例化并且调用它，因此就要在服务器中安装[-[Web]-]容器。

> [-[Web]-]容器就是一个可以运行[-[Servlet]-]的容器，并且也具备了[-[HTTP]-]服务器的功能。故[-[Web]-]容器可以简单理解为[-[HTTP]-]服务器 + [-[Servlet]-]容器。

[-[Web]-]容器是[-[Servlet]-]唯一认得的[-[HTTP]-]服务器，所以如果希望[-[Servlet]-]编写的[-[Web]-]应用程序可以正常运行，就必须知道[-[Servlet]-]如何与[-[Web]-]容器沟通、[-[Web]-]容器如何管理[-[Servlet]-]的各种对象的问题。

因为[-[Servlet]-]是[-[Java]-]类，容器需要实例化[-[servlet]-]，那么也就是说对于[-[Serlvet]-]来说[-[Web]-]容器就是一个用[-[Java]-]写的程序并且运行在[-[JVM]-]之上，它负责将[-[HTTP]-]那些文字性的通信信息，转变为[-[Serlvet]-]中可用的[-[Java]-]对象。当一个请求到来时，[-[Web]-]容器都会为每个请求分配一个线程。

![请求](/img/request-and-response.png)

**一个请求/响应的例子：**
* 客户端对[-[Web]-]服务端发出[-[HTTP]-]请求
* [-[HTTP]-]服务器收到请求，将请求交给[-[Web]-]容器处理，[-[Web]-]容器会剖析[-[HTTP请求内容]-]，创建各种对象[-[HttpServletRequest、HttpServletResponse]-]
* [-[Web]-]容器由请求的URL决定要使用哪个[-[Servlet]-]来请求
* [-[Servlet]-]根据请求对象的信息决定如何处理，通过响应对象来创建响应
* [-[Web]-]容器与HTTP服务器沟通，服务器将响应转换为[-[Http]-]相应给客户端

## [-[Servlet]-]原理

来看看相关API架构：
````java
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}
````
通过顶级的Servlet接口，我们可以了解到该顶级接口定义了[-[Servlet]-]的基本行为：与生命周期有关的[-[init、destory]-]方法，提供服务所要调用的[-[service]-]方法。我们看到[-[Srvice]-]方法的参数并不是与[-[HTTP]-]有关的，这是因为[-[Servlet]-]设计就不仅仅是为了[-[HTTP]-]协议的，其具体要它子类来实现。接下来看看常用的[-[HttpServlet]-]子类的具体实现：

````java
// httpSrevlet 中的 service方法
 protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String method = req.getMethod();
        long lastModified;
        if (method.equals("GET")) {
            .....
            this.doGet(req, resp);
        } else if (method.equals("POST")) {
            this.doPost(req, resp);
        }
            .....
    }
````
省掉一些细节，我们知道当请求来时，容器会调用[-[Servlet]-]的[-[service]-]方法，之后按照[-[HttpServlet]-]的[-[service]-]方法中定义的去判断请求方式，再分别调用相对应的方法。

所以要实现自己的[-[Servlet]-]，我们会通过继承[-[HttpServlet]-]并且重写对应方法来实现。

````java
@WebServlet("/myServlet")
public class MyServlet extends HttpServlet{
     @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       
    }
}

````
`@WebServlet`该注解表示，如果请求URL是`/mySerlet`，则调用[-[MyServlet]-]来处理该请求，如果没有设置则默认是该类完整名称。当然除了注解的方式外，我们还可以使用`web.xml`这个配置文件配置[-[url]-]与[-[servlet]-]的对应关系。

````xml
<servlet>
    <servlet-name>MyServlet</servlet-name>
    <servlet-class>com.dream.ServletDemo</servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>MyServlet</servlet-name>
    <url-pattern>/testDemo</url-pattern>
</servlet-mapping>
````
默认情况下容器会在首次请求需要某个[-[Servlet]-]服务时，才将对应的Servlet类实例化、初始化操作，但是如何想要程序启动时就初始化就要设置`load-on-startup`属性，`load-on-startup`值大于0表示应用程序启动时，就先将Servlet类载入、实例化做好初始化动作，大于零的数值就表示了[-[Servlet]-]的初始化顺序。

### URL模式设置

通过[-[URL]-]与[-[Servlet]-]相映射，容器才能根据对应的[-[URL]-]进入相对应的[-[Servlet]-]中进行处理。一个请求[-[URL]-]实际上由三个部分组成

````txt
requestURI = contextPath + servletPath + pathInfo
String requestURI = req.getRequestURI();
String contextPath = req.getContextPath();
String servletPath = req.getServletPath();
String pathInfo = req.getPathInfo();
````
之后的概念就拿访问链接`localhost:8000/Servlet_war/testDemo/servlet/test`解释。并且[-[Servlet]-]对应的访问链接为`/testDemo/srvelt/*`。

**环境变量（[-[contextPath]-]）：**容器用来决定哪个[-[Web]-]应用程序的依据（一个容器上可以部署多个[-[Web]-]应用程序）。对应上面的便是`/Servlet_war`。

![contextPath](/img/contextpath.png)
就是[intellJ搭建Java web项目](https://blog.csdn.net/gaoqingliang521/article/details/108677301)配置的第22处。

一旦决定是哪个应用来处理请求，接下来就进行[-[Servlet]-]的挑选，以下是[-[Servlet]-]设置对应[-[URL]-]的相关规则：
* 路径映射：以“/”开头但是以“/*”结尾的[-[URL]-]模式。`@WebServlet(/testDemo/servlet/*)`，则匹配到`contextPath/testDemo/serlet`的请求都会转发到该[-[Servlet]-]去处理。
* 扩展映射：以“*.”开头的[-[URL]-]模式。若[-[URL]-]模式设置为[-[*.view]-]，则所有以[-[.view]-]级为的请求都会交给该[-[Servlet]-]去处理。
* 预设[-[Servlet]-]：仅包括“/”的[-[URL]-]模式，当找不到适合的[-[URL]-]模式对应时，就会使用预设[-[Servlet]-].
* 完全匹配：不符合以上设置的其他字符串，都要做路径的严格对应。若设置“/guest/test.view”，则请求不包括请求参数部分，必须是“/guest/test.view”。

**[-[Servlet]-]路径：**[-[Servlet]-]部分指的是[-[Servlet]-]路径。不包括路径信息与请求参数，也就是与[-[Servlet]-]映射的[-[URL]-]，对应上面的便是`/testDemo/Servlet`

**路径信息：**该信息不包括请求参数，指的是不包括环境路径与[-[Servlet]-]路几个部分的额外路径信息。如果没有额外路径信息，则为[-[Null]-]。对应上面的便是`/test`.

## [-[ServletConfig、GenericServlet]-]

每个[-[Servlet]-]都必须由[-[Web]-]容器读取[-[Servlet]-]设置信息（无论是注释还是[-[web.xml]-]）、初始化之后才真正成为了一个[-[Servlet]-]。对于每个[-[Servlet]-]的设置信息，[-[Web]-]容器会为其生成一个[-[ServletConfig]-]作为代表对象，你可以从该对象中获取[-[Servlet]-]初始参数，以及表示整个应用程序的[-[ServletContext]-]对象。

![Servlet初始化生命周期](/img/life_cycle.png)

这个过程只会在创建[-[Servlet]-]实例后发生一次，之后每次请求到来，调用[-[Servlet]-]实例的[-[service]-]方法进行服务。

### [-[ServletContext]-]

[-[ServletContext]-]接口定义了运行[-[Servlet]-]应用程序环境的一些行为与观点。使得[-[ServletContext]-]实现对象来取得请求资源的[-[URL]-]、设置与存储属性、应用程序初始参数，甚至动态设置[-[Servlet]-]实例。

当整个[-[Web]-]应用程序加载[-[Web]-]容器之后，容器会产生一个[-[ServletContext]-]对象作为整个程序的代表，并设置给[-[ServletConfig]-]，只要通过[-[servletConfig]-]的[-[getServletContext]-]方法就可以取得[-[ServletContext]-]对象。

几个[-[API]-]：
* [-[getRequstDispatcher]-]：与[-[request.getRequestDispatcher]-]有啥区别？

这个就要分情况讨论了，如果参数[-[URL]-]使用的是绝对路径，这两个就没啥区别。但是如何是相对路径，这个两个方法的对应的根路径不同，[-[ServletContext]-]表示的是整个项目的根路径，而[-[Request]-]的表示当前请求的根路径。如下
````java

//假设其项目路径 localhost:8080/project/

//此Servlet的URL对映为 /dispatcher/req
request.getRequestDispatcher("/demo").include(req,resp); //结果 localhost:8080/project/demo

//此Servlet的URL对映为 /dispatcher/servlet
servletContext.getRequestDispatcher("/demo").include(req,resp); //结果 localhost:8080/project/demo


request.getRequestDispatcher("demo").include(req,resp); // localhost:8080/project/dispatcher/demo

servletContext.getRequestDispatcher("demo").include(req,resp); //相对路径会报错

````

[具体代码]()

* [-[getResourceAsStream]-]：如果想在[-[Web]-]应用程序中读取某个文件内容，就是用这个方法。使用指定路径必须以“/”作为开头，表示应用程序环境根目录，或则相对应[-[/WEB-INF/lib]-]中[-[JAR]-]文件里[-[META-INF/resources]-]的路径，返回结果会返回[-[InputStream]-]实例，接着就可以运用它来读取文件内容。



## 参考资料

* [intellJ搭建Java web项目](https://blog.csdn.net/gaoqingliang521/article/details/108677301)

指出上面博客的错误的点：
1. 21处的告警直接[-[fix]-]没用，直接到该页的Deployment配置下本项目就好了，也就是22处的操作。
2. 24处，直接点击查看父类方法就知道了，父类方法判断下客户端发起请求协议的版本，过低就会提示405。





