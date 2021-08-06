# 面向切面编程[-[AOP]-]

> 面向切面编程通过从程序结构的方面实现了横向编程，也就是在不修改主要业务逻辑代码，只是在此基础上添加一些与主要业务无关的功能，其特性补充了传统[-[OOP]-]无法解决的重复性（性能监控、事务管理、安全检查、缓存），将业务逻辑和系统处理的代码解耦。

在此之前，我们了解下[-[AOP]-]的几大基本术语。
* 切面（[-[Aspect]-]）：横穿多个类的模块。在[-[Spring AOP]-]中，切面使用`@Aspect`声明。
* 连接点（[-[Join Point]-]）：一个程序运行期间的某个时机。在[-[Spring]-]中通常表现为一个方法的调用。
* 通知（[-[Advice]-]）：声明切面的某个连接点的动作（在调用方法之前，实现日志记录等一些与主要业务无关的功能）
* 切入点（[-[PointCut]-]）：一个表达式，用于匹配对应连接点的。
* 介绍（[-[Introduction]-]）：introduction可以为原有的对象增加新的属性和方法。
* 目标对象[-[Target Object]-]：被切入点匹配的对象
* [-[AOP Proxy]-]：由[-[AOP]-]框架建立的对象，在[-[Spring]-]框架中，其代理模式有两种[-[JDK]-]动态代理和CGLIB代理
* 织入（[-[Weaving]-]）：把增强通知应用到目标对象来创建新的代理对象的过程。

![aop](/img/aop.png)

!> 实现[-[AOP]-]主要有两点：一、如何通过pointcut和advice定位到特定的joinpoint上；二、如何在advice中编写切面代码；

<details>
<summary>代码</summary>

```java
@Aspect
public class AopDemo{
    @PointCut(....)
    public void pointcut(){

    }

    @Before("pointcut()")
    public void enhanceMethod1(JoinPoint joinPoint){}

    @AfterReturning("pointcut()")
    public void enhanceMethod1(JoinPoint joinPoint){}

    @AfterThrowing("pointcut()")
    public void enhanceMethod1(JoinPoint joinPoint){}

    @After("pointcut()")
    public void enhanceMethod1(JoinPoint joinPoint){}

    @Around("pointcut()")
    public void enhanceMethod1(JoinPoint joinPoint){}
}
```

</details>

解决下问题一：如果通过[-[pointcut]-]和[-[advice]-]定位到我们需要的方法上。

切入点确定了感兴趣的接入点并且也让我们控制了[-[advice]-]什么地点运行。[-[Spring AOP]-]只支持[-[Spring bean]-]连接点的执行，也就说**切面要被声明为能够纳入[-[Spring]-]容器的组件才能运行**。一个切点声明了两个方法：一个匹配标识符其中包含一个名称、任何参数和一个切入点表达式，它确定我们需要匹配的接入点。


<details>
<summary>表达式的几种写法</summary>

* [-[args()]-]：限制连接点匹配参数为指定类型的执行方法
* [-[@args()]-]：限制连接点匹配参数由指定注解标注的方法
* [-[execution()]-]：用于匹配时连接点的执行方法
* [-[this()]-]：限制连接点匹配的[-[AOP]-]代理的bean引用为指定类型的类
* [-[target()]-]：限制连接点匹配目标对象为指定类型的类
* [-[@target()]-]：限制连接点匹配特定的执行对象，这些对象对应的类要具有指定类型的注解
* [-[within()]-]：限制连接点匹配指定的类型
* [-[@within()]-]：限制连接点匹配指定注解所标注的类型
* [-[@annotation]-]：限定匹配带有指定注解的连接点


几点要注意的：

`within()`函数定义的连接点是针对目标类而言的，而非针对运行期对象的类型而言，这一点与`execution()`是相同的。但是`within()`和`execution()`函数不同的是，`within()`所指定的连接点范围最小是类，而`execution`所指定的连接点可以大到包，小到方法入参。所以从某种意义上，`execution()`函数功能涵盖了`within`函数的功能。

`target()`切点函数通过判断**目标对象**是否按照类型匹配指定类来决定连接点是否匹配，用于匹配当前目标类型的执行方法；注意是目标对象的类型匹配，这样就不包括引入接口也类型匹配。

`this()`切点函数则通过判断**代理类**是否按类型匹配指定类来决定是否和切点匹配。用于匹配当前[-[AOP]-]代理对象类型的执行方法；注意是[-[AOP]-]代理对象的类型匹配，这样就可能包括引入接口也类型皮皮额。`this`中使用的表达式必须是类型全限定名，不支持通配符。

切点表达式之间还可以通`&&`,`||`和`!`相互组合。
```java
@Pointcut("within(com.dream.springbootexample.anno.service.AnnoServiceImpl)&&execution(* test(..))&&args(String)")
public void method(){

}
// ==》 等价于
@Pointcut("execution(* com.dream.springboxxeample.anno.service.AnnoserviceImpl.test(String))")
public void method(){

}
```

</details>

当匹配好了对应的接入点，我们还需要在`adivce`中编写增强代码，并且指定执行时机：在于且如匹配的方法执行之前、之后或是环绕执行。

* [-[Before]-]：该声明运行在接入点之前，无法阻止执行程序流继续到连接点（除非抛出异常）
* [-[After returning]-]：连接点方法执行完成，声明运行
* [-[After throwing]-]：当连接点抛出异常之后，则声明运行
* [-[After]-]：只连接存在不论是不是正常执行都会运行，相当于[-[finally]-]
* [-[Around]-]：围绕连接点的通知，方法调用

## 对传入参数的访问

当我们需要拦截目标对象的参数时，可以使用如下方法：
```java
// 获取返回值
@AfterReturning(
    pointcut = "....",
    returning = "retVal"
)
public void doAccessCheck(Object retVal){

}

@AfterThrowing(
    pointcut="",
    throwing="ex")
public void doRecoveryActions(DataAccessException ex) {
    // ...
}

// 传递参数
@Before("... && args(account)")
public void validateAccount(Account account) {
    // ...
}

```

## 获取目标对象

每个`adivce`方法都可以声明`JoinPoint`对象。该对象封装了`AOP`中切面方法的信息。

```java
@Before()
public void demo(JoinPoint joinPoint){
    // 获取目标对象的全限定名称
    joinPoint.getSignature();
    // 获取传入目标方法的参数对象
    joinPoint.getArgs();
    //获取被代理对象
    joinPoint.getTarget();
    //获取代理对象
    joinPoint.getThis();
}
```
由于`JoinPoint`是接口，需要传入的是其子类`MethodInvocationProceedingJoinPoint`。其直接父类`ProceedingJoinPoint`对象包含了
```java
//执行目标方法
Object proceed();
Object proceed(Object[] var)

```


<details>
<summary>具体用法</summary>

```java
@Around("....")
public void aroundExample(JoinPoint joinPoint){
    Object result = null;
    try{
        joinPoint.getArgs();
        MethodSignature signature =(MethodSignature)joinPoint.getSignature();
        // 当前方法
        Method method = signature.getMethod();
        //前置通知
        ...
        //执行方法
        result = joinPoint.proceed(new Object[]{});
        // 后置通知
        ...
    }catch(Throwable e){
        // 异常通知
        throw new RuntimeException(e);
    }
    // 后置通知
    ...
    return result;
}
```

</details>

## 介绍

* [target与this的区别](https://artisan.blog.csdn.net/article/details/77861658?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-2.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-2.control)其中用到了介绍。

<details>
<summary>例子</summary>

```java
// 需要添加功能的类
@Component
public class NeedEnhance{
    public void doMethod(){
        System.out.println("doMethod方法");
    }
}
// 所要添加功能
public interface IEnhance {
     void doEnHance();
}
@Component
public class Enhance implements IEnhance{
    @Override
    public void doEnHance(){
        System.out.println("增强方法");
    }
}

// 定义切面
@Aspect
@Component
public class EnhanceApsect {
    // 都是要填实现的类
    @DeclareParents(value = "com.dream.springbootexample.introduction.test.NeedEnhance",defaultImpl = Enhance.class)
    public IEnhance enhance;

    @After("this(com.dream.springbootexample.introduction.test.IEnhance)")
    public void test(){
        System.out.println("test Enhance method");
    }
}

/*
* 之后可以做 如下转换
  ((IEnhance) needEnhance).doEnHance();
*/

```

!> 需要注意的点：

*  增强的类需要接口
*  增强后使用this，添加后的接口调用不会匹配这个表达式


</details>