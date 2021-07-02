# 请求和响应

[-[Servlet]-]本质上还是[-[Java]-]对象，[-[Web]-]容器负责了对其实例的创建，完成了[-[Servlet]-]名称注册和[-[URL]-]模式的对应。当请求来到[-[HTTP]-]服务器，当服务器转交请求给容器时，容器会创建一个代表**当次请求**的[-[HttpServeltRequest]-]对象，并将相关设置信息设置给该对象，同时容器还会创建一个[-[HttpServletResponse]-]对象，作为稍后要对客户端进行响应的[-[Java]-]对象。接着容器会根据[-[URL]-]匹配对应的[-[Servlet]-]，调用它的[-[Service]-]方法，将创建的[-[HttpServletRequest]-]对象、[-[HttpServletResponse]-]对象转入作为参数，最终由容器转换为[-[Http]-]响应，再有[-[Http]-]服务器对浏览器进行响应，之后容器将[-[HttpServeltRequest]-]对象、[-[HttpServeltResponse]-]对象销毁回收。

![请求过程](/img/request_for_httpServe.png)

我们知道[-[Http]-]基于请求/响应、无状态的协议。每一次的请求/响应后，服务端会不记得任何客户端的信息，为了符合这一特性，容器每次请求都会创建新的[-[HttpServletRequest]-]和[-[HttpServletResponse]-]对象，响应后将销毁这次的请求/响应对象。也就是说下次请求时创建的请求/响应对象与上次创建的请求/响应对象无关了。

## 与[-[HttpServletRequest]-]有关的情况

### 处理请求参数与标头
* `getParameter()`：指定请求参数名称来取得对应的值
* `getHeader()`：指定标头名称后返回字符串值
等等

### 参数编码处理

对于同一个内容，如果服务器与浏览器使用不同的编码进行解析则会出现乱码现象。所以为了解决这一个问题，我们需要协调服务器和浏览器使用同一种文字编码。这需要分为[-[Post]-]与[-[GET]-]情况来说明。

**[-[Post]-]：**如果客户端没有在[-[Content-Type]-]请求头设置字符编码信息常见（[-[Content-Type:text/html;charset=UTF-8]-]）,此时使用`getCharacterEncoding`返回值为[-[null]-]。在这种情况下，容器使用[-[ISO-8859-1]-]这一编程作为默认编码处理，此时客户端使用[-[UTF-6]-]发送非[-[ASCII]-]字符的请求参数获得之后就是乱码。

````java
    //在任何请求值之前
    req.setCharacterEncoding("UTF-8");
    //或是重新设置字符集一个个解码
    URLDEcoder.decode("xxx","UTF-8");
````

**[-[Get]-]：**上面重新设置字符集的方法仅适合[-[Post]-]的请求体有效。这是因为对于GET来说，处理[-[URL]-]的时[-[HTTP]-]服务器，而非[-[Web]-]容器。相对应解决方法如下

````java
    String name = req.getParameter("name");
    String encodeName = new String(name.getBytes("ISO-8859-1"),"UTF-8");
````

### 使用[-[RequestDispatcher]-]调用请求

[-[RequestDispatcher]-]这个类是处理需要多个[-[Servlet]-]完成请求的情况。例如：将另一个[-[Servlet]-]的请求处理流程包含进来（Include）进来，或请求转发（Forward）给你的[-[Servlet]-]处理。该对象由`httpServletREquest.getRequestDispatcher()`取得。

**[-[Include]-]** 方法可以将另一个[-[Servlet]-]的操作流程包括至目前[-[Servlet]-]操作流程之中。其中设置的范围属性`setAttribue`，可以被接下来的[-[Servlet]-]访问到。

![include](/img/RequestDispatcher_include.png)

[具体代码](https://github.com/Smallart/JavaStore/tree/main/Servlet/src/main/java/com/dream/include)

**[-[Forward]-]：** 调用时同样传入请求与响应对象，这表示你要讲请求处理转发给别的[-[Servlet]-]，“对客户端的响应同时也转发给另一个[-[Servlet]-]”。

要注意的是，若要调用该方法，目前的[-[Servlet]-]不能有任何响应确认，如果在目前的[-[Servlet]-]中通过响应对象设置一些响应但未确认（响应缓冲区未满或未调用任何清除方法，则所有响应设置会被忽略），如果已经有响应确认且调用该方法，则会抛出`IllegalStateException`。

[具体代码](https://github.com/Smallart/JavaStore/tree/main/Servlet/src/main/java/com/dream/forward)

## 与[-[HttpServletResponse]-]相关
### 设置响应头、缓冲区
注意一点，所有的响应头设置，必须在响应确认之前，在响应确认之后设置的表头会被容器忽略。容器可以（但非必要）对响应进行缓冲，通常容器默认都会对响应进行缓冲。可以操作[-[HttpServletResponse]-]有关缓冲的几个方法：
````java
getBufferSize()
setBufferSize()
isCommitted()
reset()
resetBuffer()
flushBuffer()
````
[-[setBufferSize()]-]必须在调用[-[getWriter()]-]或是[-[getOutputStream]-]方法之前调用。在缓冲区未满之前，设置的响应相关内容都不会真正传至客户端，可以使用[-[isCommitted]-]看看是否确认响应。
[-[fulshBuffer]-]会清除所有的缓存区中已经设置的响应信息至客户端，[-[reset]-]和[-[resetBuffer]-]必须在响应未确认前调用。




