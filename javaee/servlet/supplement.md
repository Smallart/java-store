# 补充

> 之后遇到有关Servlet的问题都在这里进行补充记录。

## [-[Session]-] 与 [-[Cookie]-]

[-[Session]-] 与 [-[Cookie]-]存在的目的是为了帮助在以[-[HTTP]-]创建的无状态连接之上的[-[Web]-]应用能够在逻辑上保持用户的连接状态。[-[Session]-]在服务器端保存用户的相关信息，而[-[Cookie]-]在客户端保存标识该[-[Session]-]的[-[Session ID]-]。当第一次连接时，服务器会将创建好的[-[Session ID]-]放入响应头里，给[-[Web]-]浏览器创建本地[-[Cookie]-]。之后客户端发送请求时，都会将这个[-[Session ID]-]放入到请求头部，使得该次会话的[-[Session]-]都是同一个。

**[-[Cookie]-]与[-[Session]-]的不同在于：**
* 作用范围不同，一个在客户端，一个在服务器端
* 存储大小不同，单个[-[Cookie]-]保存的数据不能超过[-[4k]-]，而Session没有限制
* 有效期不同，[-[Cookie]-]默认是一次会话，关闭浏览器之后[-[Cookie]-]就会被清除，而[-[Session]-]默认的生存时间为半小时
