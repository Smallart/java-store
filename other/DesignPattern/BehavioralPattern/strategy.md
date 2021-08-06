# 策略模式（[-[Strategy]-]）

实现一个功能存在多种算法或策略，我们可以根据环境或条件的不同选择不同的算法或者策略来完成该功能。虽然我们可以使用标识和`Switch`条件语句来调用不同的策略，但是之后对于增加、删除都需要修改原代码，不易维护，违背了开闭原则。

> 策略模式：该模式定义了一系列算法，并将每个算法封装起来，使它们可以相互替换。这样把算法的责任和算法的实现分隔开来了，并委派给不同的对象对这些算法进行管理。

![Strategy](/img/strategy.gif)

<!-- panels:start -->
<!-- div:left-panel -->

**优点**
* 避免了使用多重条件语句，如`if...else if...else`或`switch`
* 策略模式提供了相同行为的不同实现，客户可以根据不同时间或空间要求选择不同
* 支持开闭原则，可以在不修改源代码的情况下，灵活增加新算法

<!-- div:right-panel -->

**缺点**
* 客户端必须理解所有策略算法的区别，以便选择恰当的算法类
* 策略模式造成了很多的策略类，增加维护难度

<!-- panels:end -->

?> [代码](https://github.com/Smallart/DesignModule/tree/80a2d0785eb37e4a6e5a9d0a4472336c3a16c0b0/StrategyPattern/src/main/java/com/smallear)