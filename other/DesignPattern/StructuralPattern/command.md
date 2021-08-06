# 命令模式（[-[Command]-]）

> 将一个请求封装为一个对象，使发出请求的责任和执行请求的责任分割开。这样两者之间通过命令对象进行沟通，这样方便将命令对象进行存储、传递、调用、增加和管理。

![Command](/img/command.gif)


<!-- panels:start -->
<!-- div:left-panel -->

**优点：**
* 通过引入中间件（抽象接口）降低系统的耦合度
* 采用命令模式增加和删除命令不会影响其他类
* 可以实现命令的撤销和恢复

<!-- div:right-panel -->

**缺点：**
* 产生大量的类，增加了系统的复杂性

<!-- panels:end -->

?> [代码](https://github.com/Smallart/DesignModule/tree/main/CommandPattern/src/main/java/com/smallart)