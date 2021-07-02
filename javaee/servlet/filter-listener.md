# 过滤器和监听器

## 监听器
[-[Web]-]容器管理着许多对象的生命周期：[-[Servlet]-]、[-[HttpRequest]-]、[-[HttpResponse]-]和[-[ServletContext]-]等等。如果我们想要在这些对象的生命周期中某些关键点（初始化、销毁和赋值操作）做些处理，我们就需要使用到[-[Web]-]提供的监听器([-[Listener]-])，当我们实现好了注册到[-[Web]-]容器后。一但时机到达之后，[-[Web]-]容器就会调用相对应的方法。

[-[Servlet]-]提供了很多接口，按照需求去实现相应接口就好了。

* [-[ServletContext]-]
    * [-[ServletContextListener]-]：应用程序生命周期监听器。监听何时Web应用程序已经初始化或即将结束销毁。
    * [-[ServletContextAttributeListener]-]：监听属性改变，当设置、移除或替换[-[ServletContext]-]属性触发。

* [-[HttpSession]-]
    * [-[HttpSessionListener]-]：会话生命周期监听器
    * [-[HttpSessionAttributeListener]-]：属性改变监听器
    * [-[HttpSessionBindingListener]-]：对象绑定监听器，setAttribute中值是一个应用对象类型
    * [-[HttpSessionActivationListener]-]：对象迁移监听器，在使用分布式环境中，应用程序的对象可能分布在多个[-[JVM]-]中，当[-[HttpSession]-]要从一个[-[JVM]-]迁移到另一个[-[JVM]-]时就会用到。

* [-[HttpServletRequest]-]
    * [-[ServletRequestListener]-]：请求生命周期监听器
    * [-[ServletRequestAttributeListener]-]：属性改变监听器

````java
    @WebListener
    public class ServletRequestListener implements ServletRequestAttributeListener {
        @Override
        public void attributeAdded(ServletRequestAttributeEvent srae) {
            
        }
        @Override
        public void attributeRemoved(ServletRequestAttributeEvent srae) {

        }

        @Override
        public void attributeReplaced(ServletRequestAttributeEvent srae) {

        }
    }
````
[具体代码](https://github.com/Smallart/JavaStore/tree/main/Servlet/src/main/java/com/dream/listeners)

## 过滤器

在容器调用[-[Servlet]-]的[-[service]-]方法前，[-[Servlet]-]并不知道请求到来，而在[-[Servlet]-]的[-[service]-]方法运行之后，容器真正对浏览器进行[-[HTTP]-]相应之前，浏览器也不知道[-[Servlet]-]真正响应是什么。而过滤器（[-[Filter]-]）正如它的名字，是介于[-[Servlet]-]之前，可以拦截过滤对[-[Servlet]-]请求，也可以改变[-[Servlet]-]对浏览器的响应。

![过滤器](/img/filter.png)

过滤器是为了满足某些独立元件（字符替换、编码设置）这类的特点是**随时可以加入应用程序中，随时可以移除并且这样不会修改原来的程序**。[-[Servlet]-]提供的过滤器还可以调整过滤顺序，也可以针对不同的[-[URL]-]应用不同的过滤器，甚至在不同Servlet间请求转发或包含是应用过滤器。

![过滤器重定向](/img/filter-direct.png)

**过滤器具体实现如下：**

````java
@WebFilter("/*")
public class FilerDemo implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        /* 放行到下一个过滤器，如果没有过滤器就调用请求目标Servlet的service方法。或则因为某个情况没有调用doFilter方法，则请求就不会继续交给接下来的过滤器或是目标Servlet。
        */
        filterChain.dofilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {

    }
}
````
[具体代码](https://github.com/Smallart/JavaStore/tree/main/Servlet/src/main/java/com/dream/filters)

可以发现，过滤器的设置与[-[Servlet]-]的设置很相似。`@WebFilter`中的[-[urlPatterns]-]表示这个过滤器匹配哪些请求。当过滤器类被载入容器时并实例化，容器会运行其中[-[init]-]方法并传入[-[FilterConfig]-]对象作为参数，但匹配到合适的[-[URL]-]之后就会调用[-[doFilter]-]方法。

### 封装器

在[-[javax.servlet.http]-]包中还提供了[-[HttpServletRequestWrapper]-]和[-[HttpServletResponseWrapper]-]封装器。这两个类是为了当你想要重写请求或是响应对象的某些方法而提供的，这样你就不需要实现所有的[-[HttpServletRequest]-]或是[-[HttpServletResponse]-]这两个类中实现的接口了。

用法如下，继承[-[HttpServletRequestWrapper]-]，重写需要的方法，通过[-[Filter]-]过滤器将容器生成[-[ServletRequest]-]作为参数传入到实现的请求类中（[-[HttpServletRequestWrapper]-]是HttpServletRequest的子类），通过过滤链向下传递。

````java
public class HttpRequestWrapperDemo extends HttpServletRequestWrapper {
    public HttpRequestWrapperDemo(HttpServletRequest request) {
        super(request);
    }    
}
````

````java
public class HttpRequestWrapperFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void destroy() {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        filterChain.doFilter(new HttpRequestWrapperDemo((HttpServletRequest) servletRequest),servletResponse);
    }
}
````
[具体代码](https://github.com/Smallart/JavaStore/tree/main/Servlet/src/main/java/com/dream/wrapper)