# 会话管理

由于[-[HTTP]-]为无状态的通信协议，服务器不会记得这次请求和下次请求之间的关系。然而有些功能是必须由多次请求来完成的。例如购物车，用户可以在多个购物页面之间采购商品，[-[Web]-]应用程序必须要有个方法来得知用户在这些网页中采购了哪些商品，这种记得此次请求与之后请求间关系的方法，就称之为会话管理。

## [-[Cookie]-]
[-[Web]-]应用程序会话管理的基本方式，就是在此次请求中，将下一次请求时服务器应知道的信息，先响应给浏览器，再由浏览器在之后的一次请求一并发送给应用程序，这样应用程序就可以“得知”多次请求的相关数据。

[-[Cookie]-]是在客户端存储信息的一种方式，服务器可以响应浏览器[-[set-cookie]-]头，浏览器收到这个标头与数值之后，它会以文件的形式存储在计算机上，这个文件就称之为[-[Cookie]-]。可以为[-[Cookie]-]设定一个存活期限，保留一些有用的信息在客户端，如果关闭浏览器再次打开浏览器并链接服务器时，这些[-[Cookie]-]仍在有限期限中，浏览器会使用[-[Cookie]-]将相关信息发送给服务器，服务器就可以得知一些先前浏览器请求的相关信息。

![cookie](/img/cookie.png)

```java
// 获取Cookie信息
Cookies[] cookeis = request.getCookies();

// 设置Cookie
response.setCookie();
```
在[-[Servlet 3.0]-]中，[-[Cookie]-]类新增了[-[setHttpOnly]-]方法，可以将[-[Cookie]-]标识为仅用于[-[HTTP]-]，在浏览器支持的情况下，这个[-[Cookie]-]不会被客户端脚本读取。

## [-[HttpSession]-]会话管理
对于[-[Servlet]-]这个动态改变页面的技术来说，如果想要进行会话管理，可以使用[-[HttpServletRequest]-]的[-[getSession]-]方法取得[-[HttpSession]-]对象。
````java
HttpSession session = request.getSession();
````
[-[HttpSession]-]上最常见使用的方法应该为[-[setAttribute]-]与[-[getAttribute]-]，与[-[HttpServeltRequest]-]的方法类似，可以让你在对象中设置及取得属性。[-[HttpSession]-]解决了浏览器与应用在会话期保留请求之间相关信息的需求。在会话期间，就可以当作[-[web]-]应用程序“记得”客户端的信息。就是在[-[Java]-]应用程序的角度来进行会话管理，而忽略[-[Http]-]无状态的事实。

虽然在逻辑上[-[Web]-]应用程序看似“记得”浏览器发出的请求，链接数个请求间的关系。但是底层连接是由无状态的[-[HTTP]-]协议建立的，那么容器是如何记录这些“关系”的？

每当一个新的请求到达[-[Web]-]容器，容器都会创建[-[HttpSession]-]对象，并且为每个对象赋予一个唯一的[-[ID]-]，称之为[-[Session ID]-]，并且处理完这次请求之后，响应会设置包含该[-[Session ID]-]的[-[Cookie]-]交给客户端。在[-[Tomcat]-]中[-[Cookie]-]的名称时[-[JSESSIONID]-]，之后的每次客户端请求都会带上这个特殊的[-[ID]-]，[-[Web]-]容器就会根据[-[Session ID]-]找到对应的[-[Session]-]对象，这样就可以保证在本次会话期间，该[-[HttpSession]-]唯一。

**需要注意的是：**如果没有为[-[Cookie]-]设置生存时间，那么默认关闭浏览器会实现。这个并不是指服务器端的[-[HttpSession]-]，这个[-[HttpSession]-]默认的生存时间为半小时，如果半小时之内没有访问的，才会被清除掉。如果想要立即清除掉这个，就需要调用[-[invalidate]-]方法，否则等到时间到了才会被容器销毁。

### 客户端禁用[-[Cookie]-]的情况
哎，你不是通过[-[Cookie/Session]-]来进行会话管理吗？如果用户禁止浏览器接收[-[Cookie]-]的功能，这下该怎么办呢？此时就需要将[-[Session ID]-]以[-[Get]-]请求发送给[-[Web]-]应用程序。

````java
//获取或是创建session
HttpSession session = req.getSession();
//这个方法就会产生带有Seesion ID 的URL
String url =  response.encodeURL("/url");
//如果客户端是禁用cookie的，则该URl = /url;JSESSIONID = ''
````
