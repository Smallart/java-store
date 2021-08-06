# 适配器模式（[-[Adapter]-]）

> 将一个类的接口转换为客户希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的类能一起工作

一般面对这样的情况：需要开发的具有某种业务功能的组件在现有的组件库中已经存在，但是它们与当前系统的接口规范不兼容，此时适配器模式就能很好的解决这一问题。

![adapter](/img/adapter.gif)

<!-- panels:start -->
<!-- div:left-panel -->

**优点：**

* 复用了现存类，程序员不需要修改原来代码而重用适配类
* 将目标类和适配者类解耦，解决了目标类和适配者接口一致的问题

<!-- div:right-panel -->

**缺点：**

* 增加代码阅读难度，降低代码可读性

<!-- panels:end -->

?> [代码](https://github.com/Smallart/DesignModule/tree/main/AdapterPattern/src/main/java/com/smallert)