# 原理

我最常接触到的应用场景是将[-[Shiro]-]整合到[-[Spring Boot]-]中去实现[-[Web]-]程序的相关安全操作。所以接下来就重点探讨下[-[Shiro]-]在[-[Web]-]的应用。

对于一个[-[Web]-]程序来说，面对从客户端发送来的请求[-[Web]-]程序会调用与[-[URL]-]相对应的方法，进行对应逻辑处理之后将结果返回给客户端，这样就形成了一次交互。为了系统的稳定和数据的安全性的进一步考虑就需要将系统的资源进行分类比如可以不需要登录访问的，需要登录访问的，需要某些特定角色才能进行访问的等等，不同的角色访问不同的资源。也就是不同的登录用户执行不同的安全策略。

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

![ShiroFilterChain](/img/Shiro-FilterChain.png)

[-[ShiroFilterFactoryBean]-]产生的[-[SpringShiroFilter]-]会被整合到项目中的过滤链中，当请求来到[-[SpringShiroFilter]-]的则会被代理执行自定义的过滤链，当自定义的过滤链执行完全之后，才会执行[-[SpringShiroFilter]-]之后的过滤器。


## [-[Shiro]-]中的[-[Filter]-]家族

![shiro-Filters](/img/shiro-filter-family.png)

从上面的继承树中可以知道[-[Shiro]-]封装了很多过滤器，但可以将其分为两大类：一、继承[-[AbstractShiroFilter]-]的这一类。二、继承自[-[AdviceFilter]-]的这一类。

* 继承自[-[AbstractShiroFilter]-]的这一类中我们会发现[-[SpringShiroFilter]-]这一熟悉的实现类，它最终会注册到[-[Servlet]-]的容器中，拦截请求并且先执行自定义的过滤器。

<details>
<summary>关键方法</summary>

````java

protected void doFilterInternal(ServletRequest servletRequest, ServletResponse servletResponse, final FilterChain chain)
        throws ServletException, IOException {

    //包装requset和response
    final ServletRequest request = prepareServletRequest(servletRequest, servletResponse, chain);
    final ServletResponse response = prepareServletResponse(request, servletResponse, chain);
    //创建 subject
    final Subject subject = createSubject(request, response);
    //noinspection unchecked
    subject.execute(new Callable() {
        public Object call() throws Exception {
            //更新Session访问时间
            updateSessionLastAccessTime(request, response);
            // 执行自定的过滤链
            executeChain(request, response, chain);
            return null;
        }
    });
}

protected void executeChain(ServletRequest request, ServletResponse response, FilterChain origChain)
        throws IOException, ServletException {
    FilterChain chain = getExecutionChain(request, response, origChain);
    chain.doFilter(request, response);
}

````

</details>

* 继承自[-[AdviceFilter]-]的这一类便是自定义的过滤器类，[-[Shiro]-]为我们实现了一些关于鉴权检测的一些过滤器，当然我们可以通过继承来自定义过滤器。

![SHiro](/img/ShiroFilterHierarchy.png)

1. [-[OncePerRequestFilter]-]：过滤重复请求
2. [-[AdviceFilter]-]：执行自定义的[-[Shiro]-]过滤链
3. [-[PathMatchingFilter]-]：匹配与[-[URL]-]相关的过滤器
4. [-[AnonymousFilter]-]：不需要权限认证直接放过。[-[AccessControlFilter]-]：要认证信息才能访问的资源
5. [-[AuthenticationFilter]-]：与认证相关的过滤器；[-[AuthorizationFilter]-]：与鉴权相关的过滤器。

[-[Shiro]-]实现的默认过滤器如下，这些过滤器类在初始化[-[FilterChainManager]-]时进行初始化。

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
