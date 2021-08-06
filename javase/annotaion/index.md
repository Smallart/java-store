# 注解[-[Annotation]-]

注解（也称之为元数据）为我们在代码中添加消息提供了一种形式化的方法，使我们可以在稍后的某个时刻更容易的使用这些数据。注解提供了[-[Java]-]无法表达，但是你需要完整表诉程序所需的信息。因此注解使得我们可以以编译器验证的格式存储程序的额外信息。注解可以生成描述符文件，甚至是新的类定义，并且有助于减轻编写“样板”代码的负担。每当创建设计重复工作的类或接口时，你通常可以使用注解来自动化和简化流程。

**声明一个注解**
注解的定义看上去很像接口的定义，事实上，它们和其他的[-[Java]-]接口一样，也会被编译为[-[Class]-]文件。
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test{
    String value() default "";
    int id();
}
```
注解的定义也需要一些元注解，描述该注解的一些属性，比如能够用在什么地方（[-[@Target]-]），在什么地方可用（[-[@Retention]-]）。
不包含任何元素的注解成为标记注解。

### 元注解

[-[Java]-]语言目前有5种标准注解，以及5种元注解。元注解用于注解其他的注解。
* @Target 表示注解可以用于哪些地方
    * CONSTRUCTOR：构造器声明
    * FIELD：字段声明
    * LOCAL_VARIABLE：局部变量声明
    * METHOD：方法声明
    * PACKAGE：包声明
    * PARAMETER：参数声明
    * TYPE：类、接口或是enum声明
* @Retention 表示注解信息保存的时长（RetentionPolicy）
    * SOURCE：注解将被编译器丢弃
    * CLASS：注解在Class文件种可用
    * RUNTIME：VM将在运行时也保留注解，因此可以通过反射机制读取注解信息
* @Documented：此注解保存在Javadoc种
* @Inherited：允许子类继承父类的注解
* @ Repeatable：允许一个注解可以被使用一次或多次

之前只是定义了注解的注部分，如果没有用于读取注解的工具，那么注解不会被注释有用。所以使用注解种一个很重要的部门就是，创建与使用注解处理器。Java扩展了反射的API用于帮你创造这类工具，同时它还提供了javac编译器钩子在编译时使用注解。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Doc {
    String value() default "";
}
public class DocTracker {
    public void tracker(AnnoDemo demo,Class clazz){
        try{
            Method method = clazz.getMethod("demo", String.class);
            Doc doc = method.getAnnotation(Doc.class);
            method.invoke(demo,doc.value());
        }catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
    }
}
```
**注解元素** 在上面的例子中，包含了String类型的注解元素，注解元素还可以是如下类型：
* 所有基本类型（int、float、boolean等）
* String
* Class
* enum
* Annotation
* 以上类型的数组

编译器对于注解元素的默认值有限定。首先元素不能有不确定的值。也就是元素要么有默认值，要么就在使用注解时提供元素。并且还有一个限制：任何非基本类型的元素，无论时在源代码声明还是在注解接口中定义默认值时，都不能使用null作为其值。

## 注解不支持继承
不能使用extends关键字来继承@interface。使用注解可以使得你能够为代码中加入元数据，而且不会导致代码杂乱并且难以阅读。使用注解能够帮助我们避免编写累赘的部署描述性文件，以及其他的生成文件。

