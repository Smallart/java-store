# 原理

我最常接触到的应用场景是将[-[Shiro]-]整合到[-[Spring Boot]-]中去实现[-[Web]-]程序的相关安全操作。所以接下来就重点探讨下[-[Shiro]-]在[-[Web]-]的应用。

对于一个[-[Web]-]程序来说，面对从客户端发送来的请求[-[Web]-]程序会调用与[-[URL]-]相对应的方法，进行对应逻辑处理之后将结果返回给客户端，这样就形成了一次交互。为了系统的稳定和数据的安全性的进一步考虑就需要将系统的资源进行分类比如可以不需要登录访问的，需要登录访问的，需要某些特定角色才能进行访问的等等，不同的角色访问不同的资源。那么如何去实现呢？

这里就要复习下[[-[Servlet]-]](/javaee/servlet/index)的相关知识点了，对于使用[-[Java]-]开发的[-[Web]-]程序来说，客户端发起的请求最后会到[-[Servlet]-]（相当于[-[Spring Boot]-]的[-[Controller]-]层中定义的方法）中进行处理和生成响应内容，但是在请求由[-[Web]-]容器交给对应[-[Servlet]-]之前，[-[Servlet]-]并不知道请求到来，而在[-[Servlet]-]之后，容器真正对浏览器进行[-[HTTP]-]响应之前，浏览器也不知道[-[Servlet]-]真正响应是什么。此时我们可以使用[-[Filter]-]（过滤器）在请求到达[-[Servlet]-]之前对请求进行检测和对应操作。[-[Shiro]-]便是通过这样一个原理并且在[-[Filter]-]之上进行了封装而且添加了很多有用的特性，使得实现对于敏感资源的控制变的简单、安全和高效。

## [-[Spring Boot]-]整合[-[Shiro]-]

接下来就利用[-[Spring Boot]-]、[-[Shiro]-]和[-[Thymeleaf]-]实现一个简单的权限控制应用。

```xml
<!-- 引入依赖 -->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.7.0</version>
</dependency>
<!--  thymeleaf于shiro的依赖 -->
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>2.0.0</version>
</dependency>
```
<!-- panels:start -->
<!-- div:title-panel -->
**整合[-[Shiro]-]**
<!-- div:left-panel -->
```java
@Bean
public ShiroFilterFactoryBean createShiroFilterFactoryBean(){
    ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
    factoryBean.setLoginUrl("/login");
    factoryBean.setSecurityManager(createSecurityManager());
    Map<String, String> filerChainMap = factoryBean.getFilterChainDefinitionMap();
    filerChainMap.put("/logout","logout");
    filerChainMap.put("/*","user");
    return factoryBean;
}
```
<!-- div:right-panel -->
[-[ShiroFilterFactoryBean]-]是整合到基于[-[Spring boot]-]的[-[Web]-]程序的关键。它实现了[-[Spring]-]定义的[-[FactoryBean]-]接口。作为一个工厂类，[-[ShiroFilterFactoryBean]-]的[-[getObjct]-]生成了一个（[-[SpringShiroFilter]-]）过滤器对象，当项目启动时会这个对象会被委托给过滤器代理最终注册到[-[Servlet]-]容器之中。

除了实现整合的功能之外，这个类的另一个重要的功能便是通过`getFilters`方法注册自定义的过滤器和通过`getFilterChainDefinitionMap`方法设定过滤器与[-[URL]-]之间的映射关系。最终实现效果如下图：
<!-- panels:end -->

![ShiroFilterFactoryBean](/img/ShiroFilterFactoryBean.png)


## [-[Shiro]-]中的[-[Filter]-]家族

![shiro-Filters](/img/shiro-filter-family.png)

从上面的继承树中可以知道[-[Shiro]-]封装了很多过滤器，但可以将其分为两大类：一、继承[-[AbstractShiroFilter]-]的这一类。二、继承自[-[AdviceFilter]-]的这一类。

继承自[-[AbstractShiroFilter]-]的这一类中我们会发现[-[SpringShiroFilter]-]这一熟悉的实现类，它最终会注册到[-[Servlet]-]的容器中，拦截请求并且先执行自定义的过滤器。继承自[-[AdviceFilter]-]的这一类便是自定义的过滤器类，[-[Shiro]-]为我们实现了一些关于鉴权和授权检测的一些过滤器，当然我们可以通过继承来自定义过滤器。

接下来我们就来了解下请求经过[-[Shiro]-]过滤链的处理过程：

首先请求先来到[-[SpringShiroFilter]-]过滤器，对于过滤器我们只需要看[-[doFilter]-]方法就好了。但是我们并没有在[-[SpringShiroFitler]-]中看到该方法，那就说明这里调用的是父类的方法，我们直接往上看就好了。最终在[-[OncePerRequestFilter]-]中找到了[-[doFilter]-]方法。

<!-- panels:start -->
<!-- div:left-panel -->
```java
 public final void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        String alreadyFilteredAttributeName = getAlreadyFilteredAttributeName();
        if ( request.getAttribute(alreadyFilteredAttributeName) != null ) {
            log.trace("Filter '{}' already executed.  Proceeding without invoking this filter.", getName());
            filterChain.doFilter(request, response);
        } else //noinspection deprecation
            if (/* added in 1.2: */ !isEnabled(request, response) ||
                /* retain backwards compatibility: */ shouldNotFilter(request) ) {
            log.debug("Filter '{}' is not enabled for the current request.  Proceeding without invoking this filter.",
                    getName());
            filterChain.doFilter(request, response);
        } else {
            // Do invoke this filter...
            log.trace("Filter '{}' not yet executed.  Executing now.", getName());
            request.setAttribute(alreadyFilteredAttributeName, Boolean.TRUE);

            try {
                doFilterInternal(request, response, filterChain);
            } finally {
                // Once the request has finished, we're done and we don't
                // need to mark as 'already filtered' any more.
                request.removeAttribute(alreadyFilteredAttributeName);
            }
        }
    }

```
<!-- div:right-panel -->
但是这个过滤器只是实现了重复请求连接的过滤并没有实现具体的逻辑，而是定义了一个`doFilterInternal`，并将这个方法交给子类去实现，自己只是在`doFilter`中调用该方法了。`doFilterInternal`有两种实现，我们先来看看[-[AbstractShiroFilter]-]中的实现.
<!-- panels:end -->

省略一些东西，来看看[-[AbstractShiroFilter]-]中的实现：
```java
protected void doFilterInternal(ServletRequest servletRequest, ServletResponse servletResponse, final FilterChain chain)
            throws ServletException, IOException {
        ……
        // 包装了request 和 response，将httpServletxxx转变为了ShiroHttpServletxxxx
        final ServletRequest request = prepareServletRequest(servletRequest, servletResponse, chain);
        final ServletResponse response = prepareServletResponse(request, servletResponse, chain);

        // 创建对象
        final Subject subject = createSubject(request, response);

        //noinspection unchecked
        subject.execute(new Callable() {
            public Object call() throws Exception {
                // 更新Session的访问时间
                updateSessionLastAccessTime(request, response);
                // 执行接下来的过滤器
                executeChain(request, response, chain);
                return null;
            }
        });
        ……
    }
```

省略[-[Shiro]-]相关对象的创建，我们先来看看[-[Shiro]-]是如何将请求交给自定义的过滤器的，追踪`executeChain`方法，我们最终在`proxyFilerChain.java`找到了实现，下面是对应实现方法。
<!-- panels:start -->
<!-- div:left-panel -->
```java
public void doFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {
    if (this.filters == null || this.filters.size() == this.index) {
        //we've reached the end of the wrapped chain, so invoke the original one:
        if (log.isTraceEnabled()) {
            log.trace("Invoking original filter chain.");
        }
        this.orig.doFilter(request, response);
    } else {
        if (log.isTraceEnabled()) {
            log.trace("Invoking wrapped filter at index [" + this.index + "]");
        }
        this.filters.get(this.index++).doFilter(request, response, this);
    }
}
```
<!-- div:right-panel -->
![executeChain](/img/executeChain.png)
<!-- panels:end -->

从上面方法可以知道，[-[Shiro]-]通过[-[SpringShiroFilter]-]这个过滤器截下请求，将请求先调用自己定义的过滤器链，当自己的过滤器链执行完成之后才执行[-[SpringShiroFilter]-]之后的过滤器。

自此整个[-[Shiro]-]的过滤器链的拦截流程就分析完了，接下来我们分析下自定义的过滤器的实现。

## 自定义[-[Shiro]-]过滤器

[-[Shiro]-]自定义的过滤器的继承树来自[-[AdviceFilter]-]这一族，为了了解其逻辑，我们从父类开始分析：
```java
// AdviceFilter
public void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain)
        throws ServletException, IOException {
        // preHandler返回当前自定义Filter的处理结果
        boolean continueChain = preHandle(request, response);
        if (log.isTraceEnabled()) {
            log.trace("Invoked preHandle method.  Continuing chain?: [" + continueChain + "]");
        }
        // 根据前一个Filter的处理结果来决定是否执行自定义过滤链的下一个过滤器
        if (continueChain) {
            executeChain(request, response, chain);
        }
        // 这个预留的，没啥具体功能
        postHandle(request, response);
        if (log.isTraceEnabled()) {
            log.trace("Successfully invoked postHandle method");
        }
}

```
进入`preHandler`方法到子类[-[PathMatchingFilter]-]的实现，这个类的作用是查询与请求[-[URL]-]相对应过滤器并且调用子类的`onPreHandle`方法，然后继续到[-[AccessControlFilter]-]中我们看到了该类对`onPreHandler`方法的实现：
````java
 public boolean onPreHandle(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
        return isAccessAllowed(request, response, mappedValue) || onAccessDenied(request, response, mappedValue);
    }
````
这个具体判断逻辑交给了子类去实现，也就是子类要实现两个逻辑：`isAccessAllowed`，该请求是否被允许，该子类该方法返回`false`，那么就到了`onAccessDenied`逻辑，如果不允许访问，则接下来该怎么处理。

### [-[Shiro]-]默认实现的过滤器

又回到[-[ShiroFilterFactoryBean]-]这个类，上面提到过我们可以通过这个类注册自定义的过滤器，并且设置相对应的映射。具体过程如下：
```java
public ShiroFilterFactoryBean createShiroFilterFactoryBean(){
    ShiroFilterFactoryBean factoryBean = new ShiroFilterFactoryBean();
    factoryBean.setLoginUrl("/login");
    factoryBean.setSecurityManager(createSecurityManager());
    Map<String, String> filerChainMap = factoryBean.getFilterChainDefinitionMap();
    Map<String, Filter> filters = factoryBean.getFilters();
    filters.put("myFilter",new MyFilter());
    filerChainMap.put("/logout","logout");
    filerChainMap.put("/demo","myFilter");
    filerChainMap.put("/*","user");
    return factoryBean;
}
```
当然[-[Shiro]-]为我们实现一些默认的过滤器，我们可以直接通过其名称来与[-[URL]-]配对使用：
````java
public enum DefaultFilter{
    anon(AnonymousFilter.class),
    authc(FormAuthenticationFilter.class),
    authcBasic(BasicHttpAuthenticationFilter.class),
    authcBearer(BearerHttpAuthenticationFilter.class),
    logout(LogoutFilter.class),
    noSessionCreation(NoSessionCreationFilter.class),
    perms(PermissionsAuthorizationFilter.class),
    port(PortFilter.class),
    rest(HttpMethodPermissionFilter.class),
    roles(RolesAuthorizationFilter.class),
    ssl(SslFilter.class),
    user(UserFilter.class),
    invalidRequest(InvalidRequestFilter.class);
}
````
