# 装饰器模式

> 在不改变现有对象结构的情况下，动态地给该对象增加一些职责。

在软件开发过程中，有时想要用一些现存的组件。这些组件完成了一些核心功能，但是需要在不该变其结构的情况下，可以动态扩展其功能。

![装饰模式](/img/decorator.png)

<!-- panels:start -->
<!-- div:left-panel -->
**优点：**
* 扩展性
* 使用不同的装饰类可以实现不同的功能
<!-- div:right-panel -->
**缺点：**
* 装饰模式会增加许多子类
<!-- panels:end -->

?> [代码](https://github.com/Smallart/DesignModule/tree/main/DecoratorPattern/src/main/java/com/smallart)