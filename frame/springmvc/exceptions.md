# 异常处理

对于[-[Web]-]应用，当请求过程中抛出异常[-[Spring MVC]-]的`DispatcherServlet`会委托一条实现于`HandlerexcpetionResolver`分责任链去处理抛出的异常，并且将其转换为**合适的应答**。[-[Spring MVC]-]默认的实现的责任链如下：
* `SimpleMappingExceptionResolver`：通过简单映射关系来决定是由哪个错误视图来处理当前的异常信息。
* `ResponseStatusExceptionResolver`：若抛出的异常类型上有`@ResponseStatus`注解，那么此任务处理器会处理，并且状态码会返回给[-[response]-]。
* `DefaultHandlerExceptionResolver`：默认处理器，能够处理标注的`Spring MVC`异常，并且把它转换为对应的应答状态码，一帮兜底。
* `ExceptionHandlerExceptionResovler`：通过在`@Controller`或是`@ControllerAdvice`中调用`@ExceptionHandler`方法来解决异常。


知道了处理方法，就需要更具处理方法去定义能够被这些处理方法处理的异常，这里有三种方式：每个类上添加`@ResponseStatus`、每个[-[Controller]-]层中专门添加被`@ExcpetionHandler`修改的方法处理异常、专门声明一个类全局处理异常。


## 处理范围：单个异常类
`@ReponseStatus`的主要作用就是改变应答的应答状态。但是当它声明在一个异常之上，当某个`Controller`层抛出了该异常，则会被`ResponseStatusExceptionResolver`捕获处理，将在`@ReponseStatus`上注解的内容装变为应答。

<details>
<summary>例子</summary>

```java

@ResponseStatus(code =  HttpStatus.INTERNAL_SERVER_ERROR,reason = "异常信息描述")
public class ResponseException extends RuntimeException{}

```

</details>


## 处理范围：某个Controller

在某个`Controller`中单独声明`@ExceptionHandler`方法来处理异常。

<details>
<summary>例子</summary>

```java
@Controller
public class ExcpetionContraller {
    @GetMapping("/exception/contr")
    public void controllerException(){
        throw new ControllerException("测试Controller层ExceptionHandler",500);
    }
    @ExceptionHandler(ControllerException.class)
    public ModelAndView handlException(Exception e){
        
    }
}
```

</details>

被`@ExceptionHandler`需要声明需要处理异常的类型，并且有请求有关的对象`HttpRequest`、`HttpSession`等等可以声明会被自动注入。当声明多个被`@ExceptionHanlder`修饰的方法时，声明的父类异常会捕获之后所有的子类异常。

## 处理范围：全局异常处理

专门声明一个被`@ControllerAdice`修饰的类，并且声明`@ExceptionHandler`修饰的方法来处理对应的异常。也就是将`Contoller`中声明的`@ExeptionHandler`处理方法规整到一个类中。

<details>
<summary>例子</summary>

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(RuntimeException.class)
    public String handlerException(Exception e){
        System.out.println("RuntimeException异常处理");
        if (e instanceof ControllerException){
            return e.getMessage();
        }
        return null;
    }

}

```

</details>
