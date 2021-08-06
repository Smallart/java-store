# [-[IOC]-]容器

[-[IOC]-]依赖反转也被称之为[-[DI]-]依赖注入。它指代的是一个对象被构造或通过工厂方式创建时，它所需要的依赖关系通过容器从构造参数或是字段的方式注入。这样的方式从根本上逆转了[-[bean]-]对象控制实例化或通过直接构造类来实现所需依赖。

`org.springframework.beans`和`org.springframework.context`包是[-[IOC]-]的基础。[-[BeanFactory]-]接口提供了管理任何类型对象的高级配置机制。`ApplicationContext`作为它的子类增加了其他的特性：
* 更加容易整合[-[Spring AOP]-]
* 消息资源处理
* 事件发布
* 应用层特殊的上下文内容，比如`webApplicationContext`，使用在[-[Web]-]应用

在[-[Spring]-]框架中，[-[IOC]-]容器管理组成应用的对象并且将他们称之为[-[bean]-]。[-[bean]-]指代的是哪些被实例化、组合并且[-[IoC]-]容器管理的对象。

`org.springframework.context.ApplicationContext`接口代表了[-[Spring IoC]-]容器并且负责实例化、配置和通过读取配置数据相进行整合。

下面的图从高维视角展示了[-[Spring]-]是如何工作的。你的项目中的类与配置元素绑定，在`ApplicationContext`被创建和初始化之后，对象之间的依赖得到满足，程序能够正常运行。

![spring工作原理](/img/container-magic.png)

* [-[Configuration Metadata]-](配置项)

配置项-可以指[-[XML]-]配置文件和[-[Java Config]-]-告诉[-[Spring]-]的容器如何去实例化一个对象，并且该对象的需要哪些依赖。

* 实例化一个容器
```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml");
//不过这样获取bean的方式，不建议在代码中直接书写
context.getBean("petStore",PetStoreService.class);
```

## [-[Bean]-]

一个[-[Spring IoC]-]容器管理着一个或多个[-[Bean]-]对象。对于容器本身，它将这些bean看作是BeanDefinition对象。BeanDefinition包含了以下的元数据：
* 真实的实现类名
* Bean的行为配置元素，比如bean在容器中的行为（范围，生命周期回调等等）
* 这个bean所需的依赖
* 其他对于新创建对象的配置

`ApplicationContext`实现也允许用户将对象手动注册到容器内，并且交给容器维护。当然这个做法不推荐。
```java
applicationContext.getBeanFactory().registerSingleton()或是registerBeanDefinition()
```

**命名Bean**：每个bean对象可以有一个或多个标识，而在容器中的bean对象需要有一个独一无二的标识。通常情况下，一个bean只有一个标识，但是如果它有多个bean对象，则其他的可以声明为别名。

在XML配置中，id则表示该bean的标识，通常情况下名字是字符串，但是它可以包含特殊字符。如果你声明bean的标识，则容器会为该bean对象产生一个标识，Spring的产生标识的规则：将类名的第一个字母小写，如果类名前两个或以上字母都是大写，则bean名称与类名一致。

## 依赖[-[Dependencies]-]

依赖解析过程:
* `ApplicationContext`([-[IoC]-]容器)创建并且初始化配置项中的所有bean对象
* 对于每个bean来说，它所需要的依赖以字段、构造函数参数等方式暴露出来。当bean对象创建之后，就会满足这个bean所需的依赖。
* 每个字段或是构造函数参数都是真实的定义或是对另一个bean的引用
* 每个字段或是构造函数参数都是转换成实际的值

bean对象的创建只有这个bean被请求时，当该bean创建时，可能会将它的依赖一起创建。

### 环形依赖
当某个bean对象使用构造器引入依赖时，它可能会陷入创建循环依赖的怪圈。也就是Class A需要通过构造函数有容器提供Class B。而Class B的初始化过程中需要注入Class A。此时CLass A和Class B就陷入了循环依赖的情况下，此时容器会抛出`BeanCurrentlyInCreationException`异常的情况。

一种解决的情况就是Class A通过Setter的方式进行依赖注入（DI），还有就是强迫将一个还未完全初始化的Bean注入到另一循环对象中。

## Bean范围

[-[IoC]-]的容器不仅管理着大量的bean对象，并且制定了bean对象的一些规则。其中包括了Bean的范围：
* singleton（单例）：bean对象默认的范围，在Spring IoC容器中是唯一的。
* prototype：Spring容器中可以包含多个bean实例
// 以下是web ioc容器才能生效的范围
* request：定义一个单例bean对象的一个生命周期是一次http请求，也就是说每个http请求都有它自己的bean对象
* session：生命周期为一次会话
* appliaction：生命周期为ServletContext
* websocket：生命周期为一次websocket

**singleton：**在程序运行期间，其IOC容器中只维护每个类的唯一实例。也就是每次请求从容器中获取的都是这个bean对象。

![单例](/img/singleton.png)

**Prototype：**非单例模式的bean范围，也就是每次请求某个类的bin都是一个新bin实例。

![prototype](/img/prototype.png)

当一个范围为Singleton的bean对象需要引入一个范围为Prototype的bea依赖时，要注意依赖在实例化的时间被解析。此时对于Singleton bean来说这个就是唯一实例。但是如果希望Singleton bean在运行时每次都能获得一个新的bean依赖。你就不能使用DI，因为着只能发生一次，也就是Singlton bean初始化的时候。

当不同范围的bean成为依赖时，比如想要相一个长时间存活的bean注入一个范围时request的bean。此时你可以选择AOP的方式注入。

## 制定Bean的性质

Spring框架提供了大量的接口让你自定义bean的性质。接口可以分为以下几类：lifecycle Callbacks、ApplicationContextAware and BeanNameAware 和 other Aware接口。

### 生命周期回调
为了自定义容器中bean的生命周期，你可以实现`InitializingBean`和`DisposableBean`接口。容器调用`afterPropertiesSet`在bean初始化之前，在bean销毁之后调用`destroy`。

之后可以使用注解实现相关功能`@Postconstruct`和`@PreDestroy`

`InitalizingBean`提供的`afterPropertiesSet`是在容器完成bean初始化操作之后调用。不过不推荐，其注解`@PostConstruct`可以实现与它一样的功能。

不过你也可以实现默认的提供的方法，效果与上面的一致
```java

class Example{
    void init(){

    }
    void dispose(){

    }
}
```

总结下，你有三个选择方式控制bean生命周期的行为
* 实现`InitializingBean`和`SidposblaeBean`回调接口
* 重写`init`和`destroy`方法
* 使用`@PostConstruct`和`@PreDestory`声明方式

当同一个bean被不同的方式实现控制bean声明周期的行为，其初始方法执行顺序如下：
* 注解`@PostConstruct`
* `afterPropertiesSet`
* `init`

销毁方式与上面一致。

## 启动与关闭回调函数
`Lifecycle`接口为每个对象(被Spring管理的对象大部分)定义了与生命周期相关的接口：

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```
当`ApplicationContext`接收到开始和停止标识，它会调用所有实现了`LifecyCycle`这个接口的内容。它通代理给`LifecycleProcessor`实现这一行为。

## 如何优雅的关闭Spring IoC容器在非web应用环境下

如果你想要优雅的关闭Spring IoC容器，你需要在JVM注册一个关闭钩子。这个钩子要保证优雅的关闭并且调用bean中的destory方法来将所有的资源释放。

```java

public static void main(final String[] args){
    ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("");
    ctx.registerShutdownHook();
}

```

### ApplicationContextAware和BeanNameAware

当`ApplicationContext`创建一个实现了`AppilcationContextAware`的对象实例，则会提供给这个bean对象关于`ApplicationContext`的引用。
```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```
通过获得Io容器，我们就可以根据bean名称获取相应的对象。但是不推荐这么做，这样做耦合了Spring代码，并且获得bean不会被IoC容器管理。还可以通过`ApplicationContextAware`方法获得文件资源，发布应用事件，以及访问`MessageSource`。

当某个类实现`org.springframework.beans.factory.BeanNameAware`接口，实现`setBeanName`方法，获取该bean在容器中设置的名称。


## 容器扩展点

通常情况下，应用开发者不需要是继承`ApplicationContext`类。但是当需要对IOC容器扩展时需要通过实现特殊的接口。

实现**BeanPostProcesssor：**接口，使得你可以重写容器对于bean实例化的逻辑、依赖解析的逻辑等等。还可以实现容器完成实例化，配置和初始化bean操作之后的逻辑。

`BeanPostProcessor`实例操作bean实例。也就是说，Spring IoC容器实例化bean对象然后`BeanPostProcessor`实例做它们的工作。

`ApplicationContext`自动发现实现了`BeanPostProcessor`接口并将其注册到容器中。`ApplicationContext`注册这些bean作为后处理器，以至它们能够在bean创建之后被调用

`BeanFacotryPostProcessor`接口语义上与`BeanPostProcessor`一致，但是有个最大不同点在于：`BeanFactoryPostProcessor`操作bean配置数据元。也就是说，Spring IoC容器让`BeeanFactoryPostProcessor`读配置数据元，在`BeanFacotryPostProcessor`处理这些数据元之后容器在开始初始化bean对象。

`FactoryBean`是一个可插入容器实例逻辑的一个点。如果你有复杂的初始化代码并且用java代码比使用XML配置更容易实现，你可以创建属于你自己的`FactoryBean`，然后将这个插入到容器中。

## 声明时配置
在Spring中如果要使用注解方式则需要做如下配置
```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

`<context:annotation-config/>`元素隐式注册了以下后处理器
* `ConfigurationClassPostProcessor`
* `AutowiredAnnotationBeanPostProcessor`
* `CommonAnnotationBeanPostProcessor`
* `PersistenceAnnotationBeanPostProcessor`
* `EventListenerMethodProcessor`

`@Required`：该注解说明被修饰的参数需要在通过显示属性值或是自动注入的方式在配置时间被填充。作用在setter方法上。用于检测一个bean的属性值在配置期间是否被赋值或是设置。如果没有赋值被会抛出BeanInitializationException异常。

`@Autowired`：其作用域为构造函数，setter或是字段上。

`@Primary`:当参数有多个bean对象可以赋值时，这就需要更加确切的注入逻辑了。一种方式是使用`@Primary`声明，这个注解暗示了某个特殊的bean应该优先作为该参数的注入。这个放在依赖上面

`@Qualifier`：这个放在需要注入的字段上，与`Primary`一起使用，说明哪个名字被使用。

当你尝试通过bean名为某个字段自动注入时，最好不要先使用`@Autorwired`，而应该使用`@Resource`。`@Resource`这个注解的语义通过name来查询这个独一无二的bean，声明的类型与匹配的过程无关。`@Autowired`通过byType来为字段注入对应的bean。当什么都没有规定的情况下`@Qualifier`与`@Resource`一致。

`@Autowired`应用范围在字段，构造函数和多参的方法上，而`@Resouce`只能应用到但单个字段和setter上。

`@Value`：注入配置属性

## Classpath路径扫描和管理组件

`@Component`是通过的协议标识修改的类声明为了可以被Spring管理的组件。而`@Service`、`@Controller`和`@Repository`则是`@Component`进一步特殊的语义，
`@ResetController`可以被认为是`@Controller`和`@ResponseBody`的组合。

对于是使用了这些注解的类，Spring会自动发现这些类并且捡起注册到ApplicationContext中去。

`@Bean`在组件中为Spring容器提供元数据。
`@Scope`为组件提供生命周期范围

当组件配自动程序扫描发现，如果对应组件没有设置名字则该bean名字就交由beanNameGenerator产生。

## 基于Java的容器配置

Spring新的Java配置支持的中间构件是`@configuration`注解的类和`@bean`注释的方法。
`@bean`注解通常去声明一个方法实例，也就是该方法返回一个新对象给Spring容器管理。
`@Configuration`声明这个类主要的方式是声明一批bean定义。

定义在`@Configuration`的`@bean`被称之为full bean而在其他的地方则被称之为 lite bean。其中最大的区别是前者的bean会被容器管理。

`bean`使用bean所需的依赖，这样会通过容器注入
```java
@Configuration
public class AppConfig{
    @Bean
    public TransferService transferService(AccountREpository accountRepository){
        return new TransferServiceImpl(accountRepository);
    }
}
```

所有被Configuraiton修饰的类都会被CGLIB动态代理，在这些动态代理生成的子类在调用父类的方法和创建新实例时都会去容器中看看所有范围的bean对象。

## 将多个配置类组合起来
`@import`
```java
@Configuration
public class ConfigA{
    @Bean
    public A a(){}
}

@Configuraiton
@Import(ConfigA.class)
public class ConfigB{
    @bean
    public B b(){}
}
```

## 环境
`Environment`接口是集合在容器中的抽象，它为应用提供了两个方面的建模。环境`profiles`和`properies`

profile是关于应用的配置bean，它只有在设置时匹配到对应的profiles文件才会被注册到容器中。而Enviroment的这个对象决定了哪种profiles当前被激活，而哪种profiles应该被默认激活。

Properties在任何应用中都扮演了重要的角色。这个角色的作用是为用户提供一个方便的服务接口，用于匹配属性源并从它们解析属性。

Bean定义profile文件在核心容器中提供了一种机制，允许在不同环境中注册不同的profile bean。
* 在开发时使用内存中的数据源，在生产环境中使用正式的数据库
* 在开发期间注册监视器结构
* 为客户A或客户B注册自定义的bean对象

`@profile`该注解让你知道这个主键是可以注册到容器中的，当特殊的profile被激活的时候。

```java
@Configuration
@Profile("development")
public class StandaloneDataConfig{
    @Bean
    public DataSource(){

    }
}
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```
```java
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development") 
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production") 
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```
当我们使用profile声明了在一些情况下使用的bean，我们需要告诉Spring哪个profile被激活了。
```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```
当然也可使用配置文件
```yml
spring.profiles.active
```

`@PropertySource`该注解提供了简单易用的机制向Spring的环境中添加`PropertySource`。
```java
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

## 为ApplicationContext（容器）添加能力

`org.springframework.beans.factory`包提供了对bean最基础的管理和维护。并且在`org.springframework.context`包中添加了继承至`BeanFactory`的`ApplicationContext`接口也扩展接口提供了很多功能。在此之前我们都是手动创建ApplicationContext,而对于Java web程序中就是依赖`ContextLoader`在启动时自动实例化`ApplictionContext`。

context包提供很多的特征：
* 通过MessageSource用i18n的方式访问信息
* 通过reousrceloader访问资源
* 通过ApplicationEventPublisher接口发布

## MessageSource i18n国际化
当`ApplicationContext`加载的时候，它会自动在context中搜索名为MessageSource的对象。如这个bean被找到了，则对以上方法调用的方法委托给Message Source。如果没有在当前容器中找到，则到其父容器中查询。还是没有则会实例化一个空的`DelegatingMessageSource`同接受之前声明方法的调用。

Spring提供了两个MessageSource的实现，`ReousrceBundleMessageSource`和`StaticMessageSource`。这两个都实现至`HierachicalMessageSource`。

## 访问资源

一个Applicaiton context是一个资源加载器，通常是加载资源对象。从本质上来说Resource是Java.net.URL功能更加丰富的版本。A Reouce可以访问包括classpath、本地文件系统、任何能够被标准URL描述的文件。

## BeanFactory
BeanFactory为Spring IOC容器提供了继承基础。`BeanFacotry`与`ApplicationContext`之间的不同。

`ApplicationContext`主要的目的就是加载配置文件、启动路径扫描、自动程序性注册bean和声明的组件类，以及注册的方法bean定义。并且
`ApplicationContext`包含了所有`beanFactory`的功能，但是如果要对bean进行完全控制，还是推荐使用`beanFactory`。
对于许多扩展的容器特性、如果注释处理和AOP代理，又或是BeanPostProcessor扩展都是不可缺少的。如果使用`beanFactory`则post-processort这些特殊的bean都是默认无法被发现的。


SPring是一个轻量级的控制反转和面向切面编程的对象容器框架。

Spring 给出扩展：
1. 在创建对象之前可以让你干点事
2. 容器初始化之前可以干点事
3. 在不同的阶段发出不同的事件 可以
4. 抽象出各个接口
5. 面向接口编程

tessseract-admin

1. 反射 动态创建对象，操作对象
2. 容器：通过new-》工厂方法-》IOC容器

注册中心：保存信息的地方，其他组件同一获取

Spring原理：
1. BeanDefinationReader
2. BeanFactory
3. Eviroment
4. BeanFactoryPostProsser
5. BeanPostProsser
6. FactoryBean

BeanFactory
1. 经历一堆xxxxAware把Bean需要的Spring主键注入调用Setxx给bean
2. BeanPostProcessor beforeinitializing
3. IntializingBean
4. init-method
5. BeanPostProccor afterXXX
6. DispoableBean 
7. Destory-Method

FactoryBean
ProxyFacotyBean
BeanNameAutoProxy
PropertySourcePaceholderConfigurer
Environment
