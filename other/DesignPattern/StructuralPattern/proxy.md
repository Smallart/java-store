# 代理模式

> 为某对象提供一种代理模式以控制对该对象的访问。即客户端通过代理间接访问该对象，从而限制、增加或修改该对象的一些特性。

由于某些原因需要给对象提供一个代理以控制对该对象的访问。此时，访问对象不适合或者不能直接引用目标对象，代理对象作为访问对象和目标对象之间的中介。

![代理模式](/img/proxy.png)

<!-- panels:start -->
<!-- div:left-panel -->
**好处：**
* 代理对象可以扩展目标对象的功能
* 代理模式能将客户端与目标对象分离，在一定程度上降低了系统的耦合度，增加了系统的可扩展性
<!-- div:right-panel -->
**缺点：**
* 代理模式会造成系统设计中类的数量增加
* 增加系统复杂度
<!-- panels:end -->

?> [代码](https://github.com/Smallart/DesignModule/tree/main/ProxyPattern/src/main/java/com/smallear)

## 动态代理

除了上面提供的静态代理外，之后还提供了动态代理。动态代理是指动态地在内存中构建代理对象，从而实现对目标对象的代理功能。

静态代理与动态代理的区别在于：
* 静态代理在编译时就已经实现，编译完成后代理类是一个实际的[-[Class]-]文件。
* 动态代理是在运行时动态生成的，即编译完成后没有实际的[-[Class]-]文件，而是在动态生成类字节码，并加载到[-[JVM]-]。

实现动态代理的方法有两种：一、`java.lang.reflect.Proxy`类提供的方法；二、cglib代理。

<details>
<summary>JDK动态代理的实现</summary>

```java
//以下方法就可以动态创建一个代理对象
ProxySubject proxySubject = Proxy.newPorxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h);
proxySubject.testProxy();

/*
    ClassLoader loader // 指定当前目标对象使用类加载器
*/
/*
    Class<?>[] interfaces // 目标对象实现的接口类型
*/
/*
    InvocationHandler h // 事件处理器 也就是该方法的事件增强功能的方法
    其中该接口包含了唯一的方法 invoke(Object proxy,Method method,Object[] args)。其中对象标识
    Object proxy代理实例
    Method 被代理的方法
    Object[] args 参数
*/
```
</details>

?> [代码](https://github.com/Smallart/DesignModule/blob/main/ProxyPattern/src/main/java/com/smallear/App.java)

[-[cglib]-]是一个第三方代码生成库，运行时在内存中动态生成一个子类对象从而实现对目标对象功能的扩展。为实现代理，需要通`MethodInterceptor`和`Enhancer`这两个类

<details>
<summary>cglib代理</summary>

```XML
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.2.5</version>
</dependency>
```

```java
/*
    定义一个拦截器。在调用目标方法时，CGLIB会回调MethodInterceptor接口方法拦截，来实现你自己的代理逻辑，类似与JDK的InvocationHandler接口。
*/
public class ProxyFactory implements MethodInterceptor{
    private Object target;
    public ProxyFactory(Object target){
        this.target = target;
    }
    public Object getProxyInstance(){
        //工具类
        Enhancer en = new Enhancer();
        //设置父类
        en.setSuperClass(target.getClass())
        //设置增强
        en.setCallback(this);
        //创建子类对象代理，可以强转为其父类
        return en.create();
    }
    @Override
    // obj表示被代理之后的对象、method代理之前的方法，args参数、proxy代理之后方法的引用
    /**
        一般是这样用的
        proxy.invokeSuper(obj,args)
    */
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        ....
        Object returnValue = method.invoke(target, args);
        ....
        return returnValue;
    }
}

```
</details>

[-[cglib]-]与[-[Java]-]提供的动态代理接口不同在于其代理对象[-[cglib]-]是不需要其有对应的接口的，其原理是[-[cglib]-]其代理类会继承目标对象，需要重写方法，所以目标对象不能为final类。但是[-[cglib]-]实现的动态代理效率更高。