# DispatcherServlet

[-[Spring MVC]-]基于[-[Servlet API]-]构建的[-[Web]-]框架。[-[Spring MVC]-]类似于其他的[-[Web]-]框架，也是按照某一个核心的`Servlet`去设计前端控制模式。也就是说在设计时将任何请求都会经过这个`Servlet`，有这个`Servlet`在按照相关的配置去将请求处理转发到对应的代理组件上。在[-[Spring]-]中该`Servlet`指的是`DispatcherServlet`。

`DispatcherServlet`毕竟是一个`Servlet`初始化之后就会自动注册到`Servlet`容器中。

## 容器继承

`DispatcherServlet`可以被看作是一个拥有它自己配置的`WebApplicationContext`。`WebApplicationContext`对`ServletContext`和`Servlet`都有连接。

## 特殊的[-[Bean]-]类型

`DispatcherServlet`委托特殊的`Bean`对象去处理请求并且生成合适的应答。这些特殊的对象都是按照[-[Spring]-]规定实现并被容器管理，但是你可以自定义它们的行为。

* HandlerMapping：处理器映射，具体由其子类实现，包含两个重要子类`RequestMappingHandlerMapping`、`SimpleUrlHandlerMapping`。
* HandlerAdapter：辅助`DispatcherServlet`执行特定的处理器
* HandlerExceptionResovler：将异常重定向到其他处理器或者显示HTML的错误界面
* ViewResolver：通过处理器返回的视图字符串查找具体的视图并渲染
* LocaleResolver，LocaleContextResolver：支持国际化页面，使用例如时区来解析本地化问题
* ThemeResolver：解析应用可用的主题，提供个性化框架
* MultipartResolver: 处理上传文件
* FlashMapManager：保存和检索输入输出的FlashMap，它可以将属性从一个请求传递到另一个请求的输入输出，一般应用在重定向中。

## Web MVC配置

`DispatcherServlet`会检查`WebApplicationContext`的每一个特殊的bean。如果没有对应的bean类型，则会使用`Dispatcherservlet.properties`的默认类型。

## 流程

`DispatcherServlet`请求流程如下：
* 